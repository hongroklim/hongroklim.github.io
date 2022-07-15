---
date: 2021-03-20 22:56:00 +0900
title: 데비안에 펌웨어 설치하기
excerpt: 네트워크도 안 되는데 펌웨어를 설치할 일이 생겼다
categories: dev
---

처음 데비안 설치하고 켰는데 부팅이 제대로 되질 않는다.
찾아보니 펌웨어를 설치하라고 하네.

## 문제해결의 실마리를 찾기까지

### 화면이 왜이래

별다른 문제없이 데비안(Debian) 설치를 마무리하고 처음 부팅을 하니 화면 해상도가 낮게 보였다. 그리고 콘솔에서는 몇 번 오류를 뱉어내더니 끝내
멈춰버렸다. 이런. 몇 번이고 다시 부팅을 해 봐도 똑같았고 당황한 나는 그냥 다시 설치해야 하나 잠시 고민했다. 그래도 Recovery Mode로
부팅이 되긴 하니 일단 지금 상태에서 고쳐보기로 했다.

### 잘 알려진 문제였군

[데비안 설치가이드](https://www.debian.org/releases/stable/i386/install.pdf.ko) 를 꼼꼼이 읽던 중 내 상황인 것 같은 부분을 발견했다.

<figure>
  <img src="https://i.imgur.com/bGS6M9F.png"
       alt="content_01">
  <figcaption>6.4 없는 펌웨어 읽어들이기&emsp;/&emsp;<a href="https://www.debian.org/releases/stable/i386/install.pdf.ko">출처 : 데비안 GNU/리눅스 설치 안내서 69p</a></figcaption>
</figure>

심한 번역체라서 무슨 언어추리 문제를 푸는 것 같았지만 그래도 내가 필요한 정보는 유추했다. 
> debian-installer 안에 `radeon 드라이버`가 없으므로, 이를 이용하는 하드웨어(`모니터`)는 작동이 안 될 것이다.

지금 나는 외장그래픽카드로 radeon을 사용하고 있으니 이 문제가 맞는 것 같다. 그럼 이제 radeon 드라이버를 설치하면 해결되겠다.

### 네트워크 안 되는데 어쩌지

펌웨어를 설치하는 방법은 간단하다. `/etc/apt/sources.list`에 `non-free` 항목 넣고 `apt-get`으로 설치하면 끝난다.
하지만 나는 상황이 다르다. Recovery Mode로만 부팅이 가능하기 때문에 네트워크가 설정되어 있지 않기 때문이다. 설치 전
필요한 펌웨어를 오프라인으로 옮겨두는 것 부터 먼저 해야한다.

## 설치과정

### 펌웨어 다운로드 및 USB 복사

다른 컴퓨터에서 펌웨어를 다운로드 받아 가져온다.

1. [데비안 펌웨어 이미지](https://cdimage.debian.org/cdimage/unofficial/non-free/firmware/) 중 필요한 버전 찾기 (Debian 10은 buster)
1. `current/`에 있는 `firmware.tar.gz` 다운로드
1. USB에 zip 파일 복사

### USB 마운트 및 복사

```bash
# 1. Recovery Mode 부팅 후 USB 연결

# 2. Device에서 USB 찾기 (Type과 용량으로 찾기)
$ sudo fdisk -l
# 예) /dev/sda3

# 3. USB 마운트 및 파일 복사
$ sudo mkdir /tmp/usb # 먼저 디렉토리를 만든다
$ sudo mount /dev/sda3 /tmp/usb # 원하는 디렉토리로 Device를 마운트 한다
$ sudo cp /tmp/usb/firmware.tar.gz /var/ # USB에서 데비안으로 복사한다

# 4. USB 마운트 해제
$ sudo unmount /dev/sda3
```

### 펌웨어 압축해제 및 설치

```bash
# 1. tar 파일 압축해제
$ sudo tar -xzvf /tmp/firmware.tar.gz

# 2. 펌웨어 설치 (총 4개)
$ sudo dpkg -i /tmp/firmware/firmware-amd-graphics.deb
$ sudo dpkg -i /tmp/firmware/libgl1-mesa-dri.deb
$ sudo dpkg -i /tmp/firmware/libglx-mesa0.deb
$ sudo dpkg -i /tmp/firmware/xserver-xorg-video-all.deb
# 중간에 의존성 문제가 나와도 무시한다 (어차피 다 충족시켜줄 예정)

# 3. 펌웨어 설치확인
$ ls -laR /lib/firmware/ | grep radeon
# 곳곳에 설치된 radeon들을 보기만하면 된다
```

이후 재부팅하면 정상적으로 Debian GUI로 부팅되는 것을 볼 수 있다.

## 참고자료
- [Debian Wiki - Firmware](https://wiki.debian.org/AtiHowTo#Firmware)
- [firmware-amd-graphics](https://packages.debian.org/buster/firmware-amd-graphics)
