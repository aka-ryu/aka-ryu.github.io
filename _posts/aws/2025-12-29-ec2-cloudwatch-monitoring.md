---
layout: post
title: AWS - EC2 CloudWatch 모니터링과 Google Chat 알림 설정
categories: AWS
---

## 개요

프로덕션 환경에서 안정적인 서비스 운영을 위해 EC2 인스턴스의 **CPU, 메모리, 디스크, 상태 검사**를 모니터링하고,  
문제 발생 시 **Google Chat**으로 실시간 알림을 받도록 구성하였다.

기본 CloudWatch 메트릭만으로는 메모리나 디스크 사용률을 확인할 수 없어, **CloudWatch Agent**를 설치하여  
커스텀 메트릭을 수집하고, **SNS + Lambda**를 통해 Google Chat으로 알림을 전송하는 시스템을 구축했다.

## 필요한 이유

* **CPU 과부하**: 급격한 트래픽 증가나 프로세스 문제로 CPU 사용률이 급증할 수 있음
* **메모리 부족**: 메모리 누수나 프로세스 증가로 인한 OOM(Out of Memory) 상황 방지
* **디스크 공간**: 로그 파일이나 데이터 축적으로 인한 디스크 풀 상황 방지  
* **인스턴스 상태**: 시스템 체크나 인스턴스 체크 실패로 인한 장애 상황 감지

기존에는 장애가 발생해도 늦게 인지하는 경우가 많았는데, 실시간 알림을 통해 **빠른 대응**이 가능해졌다.

---

## 1. CloudWatch Agent 설치 및 설정

### Agent 설치
```bash
# CloudWatch Agent 다운로드
wget https://amazoncloudwatch-agent.s3.amazonaws.com/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm

# 설치
sudo rpm -U ./amazon-cloudwatch-agent.rpm
```

### Agent 설정 파일 생성
메모리와 디스크 사용률을 수집하도록 설정했다.

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "cwagent"
  },
  "metrics": {
    "namespace": "CWAgent",
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          "used_percent"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "/"
        ]
      }
    }
  }
}
```

### IAM 역할 연결
EC2 인스턴스에 **CloudWatchAgentServerPolicy** 권한이 있는 IAM 역할을 연결해야 한다.

```bash
# Agent 시작
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
```

---

## 2. SNS 토픽 생성

알림을 통합 관리하기 위한 SNS 토픽을 생성했다.

```bash
# SNS 토픽 생성
aws sns create-topic --name EC2-Alerts-Topic --region ap-northeast-2
```

생성된 토픽 ARN을 기록해둔다: `arn:aws:sns:ap-northeast-2:계정번호:EC2-Alerts-Topic`

---

## 3. Lambda 함수로 Google Chat 연동

### Lambda 함수 생성
SNS 메시지를 받아서 Google Chat 웹훅으로 전송하는 Lambda 함수를 생성했다.

```javascript
import https from 'https';
import { EC2Client, DescribeInstancesCommand } from "@aws-sdk/client-ec2";

const GOOGLE_CHAT_WEBHOOK_URL = "https://chat.googleapis.com/v1/spaces/웹훅주소";

const ec2Client = new EC2Client({ region: 'ap-northeast-2' });

export const handler = async (event) => {
  console.log('SNS 이벤트:', JSON.stringify(event, null, 2));
  
  for (const record of event.Records) {
    if (record.Sns) {
      await handleSnsMessage(record.Sns);
    }
  }
  
  return { statusCode: 200, body: 'Success' };
};

async function handleSnsMessage(snsMessage) {
  try {
    const message = JSON.parse(snsMessage.Message);
    
    // 알림 타입 확인
    if (message.AlarmName && message.NewStateValue === 'ALARM') {
      await sendGoogleChatMessage(message);
    }
  } catch (error) {
    console.error('SNS 메시지 처리 중 오류:', error);
  }
}

async function sendGoogleChatMessage(alarmData) {
  // 알람 타입에 따른 이모지 설정
  let emoji = '⚠️';
  if (alarmData.AlarmName.includes('CPU')) emoji = '🔥';
  else if (alarmData.AlarmName.includes('Memory')) emoji = '💾';
  else if (alarmData.AlarmName.includes('Disk')) emoji = '💽';
  else if (alarmData.AlarmName.includes('Status')) emoji = '❌';
  
  // 인스턴스 이름 조회
  let instanceName = '알 수 없음';
  const instanceId = alarmData.AlarmName.match(/i-[0-9a-f]+/)?.[0];
  
  if (instanceId) {
    try {
      const command = new DescribeInstancesCommand({
        InstanceIds: [instanceId]
      });
      const response = await ec2Client.send(command);
      
      const instance = response.Reservations[0]?.Instances[0];
      const nameTag = instance?.Tags?.find(tag => tag.Key === 'Name');
      instanceName = nameTag?.Value || instanceId;
    } catch (error) {
      console.error('인스턴스 정보 조회 실패:', error);
      instanceName = instanceId || '알 수 없음';
    }
  }
  
  const chatMessage = {
    text: `${emoji} *AWS EC2 알림*\n\n` +
          `• *알림*: ${alarmData.AlarmName}\n` +
          `• *인스턴스*: ${instanceName}\n` +
          `• *상태*: ${alarmData.NewStateValue}\n` +
          `• *사유*: ${alarmData.NewStateReason}\n` +
          `• *시간*: ${new Date(alarmData.StateChangeTime).toLocaleString('ko-KR', { timeZone: 'Asia/Seoul' })}`
  };
  
  await sendToGoogleChat(chatMessage);
}

function sendToGoogleChat(message) {
  return new Promise((resolve, reject) => {
    const postData = JSON.stringify(message);
    const url = new URL(GOOGLE_CHAT_WEBHOOK_URL);
    
    const options = {
      hostname: url.hostname,
      path: url.pathname + url.search,
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(postData)
      }
    };
    
    const req = https.request(options, (res) => {
      let responseData = '';
      res.on('data', (chunk) => {
        responseData += chunk;
      });
      res.on('end', () => {
        console.log('Google Chat 응답:', responseData);
        resolve();
      });
    });
    
    req.on('error', (error) => {
      console.error('Google Chat 요청 실패:', error);
      reject(error);
    });
    
    req.write(postData);
    req.end();
  });
}
```

### Lambda 권한 설정
Lambda 함수가 EC2 정보를 조회할 수 있도록 IAM 역할에 권한을 추가했다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances"
            ],
            "Resource": "*"
        }
    ]
}
```

### SNS 토픽과 Lambda 연결
```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-northeast-2:계정번호:EC2-Alerts-Topic \
  --protocol lambda \
  --notification-endpoint arn:aws:lambda:ap-northeast-2:계정번호:function:ec2-alerts-to-googlechat
```

---

## 4. CloudWatch 알람 설정

### CPU 사용률 알람
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-CPU-High-i-인스턴스ID" \
  --alarm-description "CPU 사용률이 70% 이상일 때 알림" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 70 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:ap-northeast-2:계정번호:EC2-Alerts-Topic \
  --dimensions Name=InstanceId,Value=i-인스턴스ID
```

### 메모리 사용률 알람
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-Memory-High-i-인스턴스ID" \
  --alarm-description "메모리 사용률이 80% 이상일 때 알림" \
  --metric-name mem_used_percent \
  --namespace CWAgent \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:ap-northeast-2:계정번호:EC2-Alerts-Topic \
  --dimensions Name=InstanceId,Value=i-인스턴스ID
```

### 디스크 사용률 알람  
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-Disk-High-i-인스턴스ID" \
  --alarm-description "디스크 사용률이 85% 이상일 때 알림" \
  --metric-name disk_used_percent \
  --namespace CWAgent \
  --statistic Average \
  --period 300 \
  --threshold 85 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:ap-northeast-2:계정번호:EC2-Alerts-Topic \
  --dimensions Name=InstanceId,Value=i-인스턴스ID Name=device,Value=/ Name=fstype,Value=ext4 Name=path,Value=/
```

### 인스턴스 상태 검사 알람
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-StatusCheck-Failed-i-인스턴스ID" \
  --alarm-description "EC2 상태 검사 실패 시 알림" \
  --metric-name StatusCheckFailed \
  --namespace AWS/EC2 \
  --statistic Maximum \
  --period 300 \
  --threshold 0 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:ap-northeast-2:계정번호:EC2-Alerts-Topic \
  --dimensions Name=InstanceId,Value=i-인스턴스ID
```

---

## 5. Google Chat 웹훅 설정

1. Google Chat에서 알림을 받을 스페이스 선택
2. 스페이스 설정 → 앱 및 통합 관리 → 웹훅 추가
3. 웹훅 이름 입력 후 저장
4. 생성된 웹훅 URL을 Lambda 함수의 환경변수나 코드에 설정

---

## 결과

이제 EC2 인스턴스에 문제가 발생하면 Google Chat으로 다음과 같은 형태의 알림을 받을 수 있다:

```
🔥 AWS EC2 알림

• 알림: EC2-CPU-High-i-1234567890abcdef0  
• 인스턴스: 프로덕션-서버
• 상태: ALARM
• 사유: Threshold Crossed: 2 out of the last 2 datapoints [85.0 (날짜)] were greater than the threshold (70.0)
• 시간: 2025/1/1 오후 3:45:23
```

**모니터링 대상**:
- CPU 사용률 70% 이상
- 메모리 사용률 80% 이상  
- 디스크 사용률 85% 이상
- EC2 상태 검사 실패

이를 통해 장애 상황을 빠르게 인지하고 대응할 수 있어, 서비스 안정성이 크게 향상되었다.  
특히 야간이나 주말에도 실시간으로 알림을 받을 수 있어 24시간 모니터링이 가능해졌다.
