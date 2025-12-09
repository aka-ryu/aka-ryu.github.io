---
layout: post
title: ISSUE - EC2 스토리지 용량 초과로 인한 서버 다운
category: ISSUE
---

## 개요

EC2 인스턴스의 스토리지 용량이 100% 사용되면서 서버가 다운되었고, SSH 접속조차 불가능한 상황이 발생하였다.  
디스크 용량이 꽉 차면서 시스템이 정상적으로 동작하지 않아 서비스가 완전히 중단되었다.

## 원인

서버에서 로그 파일이 무한정 쌓이거나 캐시 데이터가 정리되지 않아 EBS 볼륨의 용량이 점진적으로 증가하다가 100%에 도달한 것이 원인이었다.  
용량이 꽉 차면서 시스템이 쓰기 작업을 할 수 없게 되었고, 이로 인해 SSH 데몬을 포함한 대부분의 서비스가 제대로 동작하지 않았다.

## 해결방안

### 1. EBS 볼륨 분리
EC2 콘솔에서 문제가 발생한 인스턴스를 중지(stop)한 후, 연결된 EBS 볼륨을 분리(detach)하였다.

### 2. 임시 EC2 인스턴스 생성
새로운 EC2 인스턴스를 생성하여 분리한 EBS 볼륨을 추가 볼륨으로 연결(attach)하였다.  
이를 통해 기존 데이터에 접근할 수 있게 되었다.

### 3. EBS 볼륨 용량 확장
AWS 콘솔에서 EBS 볼륨의 용량을 확장(modify volume)하였다.  
예를 들어 기존 30GB 볼륨을 50GB로 증설하는 식으로 처리하였다.

### 4. 파일시스템 확장
임시 EC2 인스턴스에 접속하여 볼륨을 마운트한 후, 파일시스템을 확장하였다.

```bash
# 볼륨 마운트
sudo mount /dev/xvdf1 /mnt/recovery

# 파티션 확장
sudo growpart /dev/xvdf 1

# 파일시스템 확장 (ext4의 경우)
sudo resize2fs /dev/xvdf1

# 또는 XFS의 경우
sudo xfs_growfs /mnt/recovery
```

### 5. 불필요한 파일 정리
마운트된 볼륨에서 불필요한 로그 파일과 캐시 데이터를 정리하여 여유 공간을 확보하였다.

```bash
# 용량 큰 파일 찾기
sudo du -h /mnt/recovery | sort -rh | head -20

# 오래된 로그 파일 삭제
sudo rm -rf /mnt/recovery/var/log/*.log.*
sudo rm -rf /mnt/recovery/tmp/*
```

### 6. 볼륨 재연결
정리가 완료된 EBS 볼륨을 임시 EC2에서 분리하고, 원래 EC2 인스턴스에 다시 연결(attach)하였다.  
기존 인스턴스를 시작(start)하여 정상적으로 부팅되는지 확인하였다.

