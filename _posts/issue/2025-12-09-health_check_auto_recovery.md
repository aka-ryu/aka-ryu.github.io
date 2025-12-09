---
layout: post
title: ISSUE - 서버 다운 시 헬스체크 자동 복구 시스템 구축
category: ISSUE
---

## 개요

서버가 지속적으로 다운되는 문제가 발생하였으나, 당장 원인 파악을 위해 코드를 상세히 분석할 시간이 없는 상황이었다.  
서비스 중단을 최소화하기 위해 임시 대응책이 필요했다.

다행이도 ec2 다운이 아닌 단순 Pm2 프로세스만 다운되는 상태

## 원인

정확한 원인은 파악하지 못했으나, 서버가 간헐적으로 응답하지 않거나 프로세스가 종료되는 현상이 반복적으로 발생하였다.  
근본적인 원인 분석보다는 우선 서비스 가용성을 확보하는 것이 급선무였다.

## 해결방안

헬스체크 스크립트를 작성하여 서버 상태를 주기적으로 모니터링하고, 다운 시 PM2를 통해 자동으로 재시작하도록 구성하였다.

### 1. 헬스체크 엔드포인트 추가
애플리케이션에 헬스체크용 엔드포인트를 추가하였다.

```javascript
// NestJS 예시
@Controller('health')
export class HealthController {
  @Get()
  check() {
    return {
      status: 'ok',
      timestamp: new Date().toISOString(),
    };
  }
}
```

```javascript
// Express 예시
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'ok',
    timestamp: new Date().toISOString(),
  });
});
```

### 2. 헬스체크 자동 복구 스크립트 작성
서버 상태를 확인하고 다운 시 PM2로 재시작하는 bash 스크립트를 작성하였다.

```bash
#!/bin/bash
# health-check.sh

# 설정
HEALTH_URL="http://localhost:3000/health"
PM2_APP_NAME="your-app-name"
LOG_FILE="/var/log/health-check.log"

# 헬스체크 실행
response=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 $HEALTH_URL)

if [ $response -ne 200 ]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Health check failed. Response code: $response" >> $LOG_FILE
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Restarting application with PM2..." >> $LOG_FILE
    
    # PM2로 애플리케이션 재시작
    pm2 restart $PM2_APP_NAME
    
    # 재시작 후 상태 확인
    sleep 5
    pm2 status $PM2_APP_NAME >> $LOG_FILE
    
    # 슬랙 알림 (선택사항)
    # curl -X POST -H 'Content-type: application/json' \
    #   --data "{\"text\":\"⚠️ Server auto-recovered at $(date)\"}" \
    #   YOUR_SLACK_WEBHOOK_URL
else
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Health check passed" >> $LOG_FILE
fi
```

### 3. 스크립트 실행 권한 부여 및 배치
스크립트에 실행 권한을 부여하고 적절한 위치에 배치하였다.

```bash
# 스크립트 디렉토리 생성
sudo mkdir -p /home/ubuntu/scripts

# 스크립트 파일 생성 및 권한 부여
sudo chmod +x /home/ubuntu/scripts/health-check.sh

# 로그 디렉토리 생성
sudo mkdir -p /var/log
sudo touch /var/log/health-check.log
sudo chmod 666 /var/log/health-check.log
```

### 4. Crontab 등록
5분마다 헬스체크를 실행하도록 crontab에 등록하였다.

```bash
# crontab 편집
crontab -e

# 아래 내용 추가 (5분마다 실행)
*/5 * * * * /home/ubuntu/scripts/health-check.sh

# 또는 더 자주 체크하고 싶다면 (2분마다)
*/2 * * * * /home/ubuntu/scripts/health-check.sh
```

### 5. 로그 로테이션 설정
헬스체크 로그가 무한정 쌓이지 않도록 logrotate를 설정하였다.

```bash
# /etc/logrotate.d/health-check 파일 생성
sudo vi /etc/logrotate.d/health-check

# 아래 내용 추가
/var/log/health-check.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
```

### 6. 동작 확인
스크립트가 정상적으로 동작하는지 수동으로 테스트하였다.

```bash
# 헬스체크 스크립트 직접 실행
/home/ubuntu/scripts/health-check.sh

# 로그 확인
tail -f /var/log/health-check.log

# cron 실행 확인
sudo grep CRON /var/log/syslog
```

이를 통해 서버가 다운되더라도 5분 이내에 자동으로 복구되도록 구성하여, 서비스 중단 시간을 최소화할 수 있었다.  
향후 시간이 확보되면 근본 원인을 분석하여 영구적인 해결책을 마련할 예정이다.

