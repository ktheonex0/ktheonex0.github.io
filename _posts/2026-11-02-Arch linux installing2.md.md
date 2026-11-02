---
layout: single
title: "Arch Linux 완벽 설치 가이드"
date: 2026-11-02 13:15:00 +0900
---

# Arch Linux 수동 설치 완벽 상세 가이드

본 가이드는 Josean Martinez의 유튜브 영상 ["The Only Arch Linux Installation Guide You'll Ever Need"](https://www.youtube.com/watch?v=TS1ghG3c3xI)를 바탕으로 작성된 아치 리눅스(Arch Linux) 수동 설치 매뉴얼입니다. 자동 설치 스크립트(`archinstall`)를 사용하지 않고 운영체제의 밑바닥부터 직접 구축하는 방법을 다룹니다.

---

## 1단계: 설치 미디어 준비 및 부팅

### 1. ISO 다운로드 및 USB 플래싱
1. [Arch Linux 공식 홈페이지](https://archlinux.org/download/)에서 최신 ISO 파일을 다운로드합니다.
2. **Balena Etcher** 또는 **Rufus**와 같은 프로그램을 사용하여 텅 빈 USB 드라이브에 ISO 이미지를 구워 부팅 가능한 USB를 만듭니다.

### 2. 라이브 환경(Live Environment) 부팅
1. 컴퓨터를 재부팅하고 BIOS/UEFI 설정으로 진입하여 **부팅 순서(Boot Order)**를 USB 드라이브 최우선으로 변경합니다.
2. Arch Linux 라이브 환경으로 부팅되면 검은색 터미널(루트 프롬프트 `root@archiso ~ #`)이 나타납니다.
3. **UEFI 모드 확인:** 시스템이 UEFI 모드로 부팅되었는지 확인합니다.
   ```bash
   ls /sys/firmware/efi/efivars
   ```
   *수많은 파일 목록이 출력된다면 정상적으로 UEFI 모드로 부팅된 것입니다.*

---

## 2단계: 네트워크 연결 및 시간 동기화

운영체제 패키지를 다운로드하기 위해 인터넷 연결이 필수적입니다.

### 1. 인터넷 연결 확인
이더넷(유선 랜)을 사용 중이라면 대부분 자동으로 연결됩니다. 아래 명령어로 확인합니다.
```bash
ping -c 3 archlinux.org
```

### 2. 와이파이(Wi-Fi) 연결 (무선 환경인 경우)
`iwctl` 도구를 사용하여 와이파이에 연결합니다.
```bash
iwctl
# iwctl 프롬프트 진입 후
device list              # 무선 랜카드 이름 확인 (예: wlan0)
station wlan0 scan       # 주변 와이파이 스캔
station wlan0 get-networks # 스캔된 와이파이 목록 출력
station wlan0 connect "와이파이이름" # 연결 시도 및 비밀번호 입력
exit                     # iwctl 종료
```
다시 `ping -c 3 archlinux.org`를 입력해 연결을 확인합니다.

### 3. 시스템 시간 업데이트
```bash
timedatectl set-ntp true
```

---

## 3단계: 디스크 파티셔닝

가장 중요하고 주의해야 할 단계입니다. 운영체제가 설치될 디스크를 나눕니다.

### 1. 디스크 식별
```bash
lsblk
```
설치할 디스크의 이름(예: `/dev/sda` 또는 `/dev/nvme0n1`)을 확인합니다.

### 2. 파티션 생성 (cfdisk 사용)
직관적인 TUI 도구인 `cfdisk`를 사용합니다. (여기서는 디스크가 `/dev/sda`라고 가정합니다.)
```bash
cfdisk /dev/sda
```
* **Label type:** `gpt`를 선택합니다.
* 기존 파티션이 있다면 모두 지운(Delete) 후, 다음 3개의 파티션을 생성(New)합니다.

| 파티션 목적 | 크기 | 파티션 타입 (Type) |
| :--- | :--- | :--- |
| **1. EFI 시스템 (부트로더)** | 500MB ~ 1GB | EFI System |
| **2. Swap (가상 메모리)** | RAM 용량과 비슷하게 (예: 4G~8G) | Linux swap |
| **3. Root (OS 및 데이터)** | 나머지 전체 공간 | Linux filesystem |

설정이 끝났다면 `Write`를 선택하여 `yes`를 입력해 저장하고 `Quit`로 빠져나옵니다. `lsblk`를 다시 입력해 파티션(예: `sda1`, `sda2`, `sda3`)이 잘 생성되었는지 확인합니다.

---

## 4단계: 파티션 포맷 및 마운트

생성된 파티션에 파일 시스템을 입히고 인식시킵니다.

### 1. 포맷 (Formatting)
* **EFI 파티션 (sda1):** FAT32로 포맷
  ```bash
  mkfs.fat -F 32 /dev/sda1
  ```
* **Swap 파티션 (sda2):** Swap으로 포맷
  ```bash
  mkswap /dev/sda2
  ```
* **Root 파티션 (sda3):** ext4 (또는 btrfs)로 포맷
  ```bash
  mkfs.ext4 /dev/sda3
  ```

### 2. 마운트 (Mounting)
* **Root 파티션 마운트:**
  ```bash
  mount /dev/sda3 /mnt
  ```
* **EFI 파티션 마운트:** 부트 디렉토리를 만들고 마운트합니다.
  ```bash
  mount --mkdir /dev/sda1 /mnt/boot
  ```
* **Swap 활성화:**
  ```bash
  swapon /dev/sda2
  ```

---

## 5단계: 베이스 시스템 설치

이제 빈 디스크에 Arch Linux의 핵심 뼈대를 다운로드하여 설치합니다.

### 1. 패키지 설치 (`pacstrap`)
```bash
pacstrap -K /mnt base linux linux-firmware base-devel nano
```
* `base`: Arch 리눅스 기본 시스템
* `linux`, `linux-firmware`: 리눅스 커널 및 하드웨어 펌웨어
* `base-devel`: 빌드 도구 (AUR 사용 시 필수)
* `nano`: 텍스트 편집기 (vim을 선호하면 vim 설치)

*설치에 다소 시간이 소요됩니다.*

---

## 6단계: 시스템 기본 설정 (Chroot)

설치된 디스크의 시스템 내부로 진입하여 설정을 진행합니다.

### 1. fstab 생성
부팅 시 파티션 마운트 정보를 시스템에 기록합니다.
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
*(권장) `cat /mnt/etc/fstab`으로 잘 기록되었는지 확인합니다.*

### 2. Chroot 환경 진입
라이브 USB가 아닌 방금 설치한 디스크의 시스템으로 접속합니다.
```bash
arch-chroot /mnt
```

### 3. 시간대 (Timezone) 설정
```bash
ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
hwclock --systohc
```

### 4. 로캘 (언어) 설정
1. `nano /etc/locale.gen` 명령어로 파일을 엽니다.
2. `#en_US.UTF-8 UTF-8` (영어) 및 `#ko_KR.UTF-8 UTF-8` (한국어) 앞의 `#`을 지워 주석을 해제하고 저장(`Ctrl+O`, `Enter`, `Ctrl+X`)합니다.
3. 변경 사항을 생성 및 적용합니다.
   ```bash
   locale-gen
   echo "LANG=en_US.UTF-8" > /etc/locale.conf
   ```
   *(참고: 콘솔(TTY) 환경에서는 한글 폰트가 깨질 수 있으므로 초기 설정은 영어(en_US)로 하는 것을 권장합니다.)*

### 5. 호스트네임 (PC 이름) 및 네트워크 설정
```bash
echo "myarchpc" > /etc/hostname  # 'myarchpc' 대신 원하는 이름 지정
```
**네트워크 매니저 설치 및 활성화:**
```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

---

## 7단계: 사용자 계정 및 권한 설정

### 1. Root(최고 관리자) 비밀번호 설정
```bash
passwd
```

### 2. 일반 사용자 계정 생성
항상 Root로 작업하는 것은 위험하므로 일상용 계정을 만듭니다. (예: `josean`)
```bash
useradd -m -G wheel josean
passwd josean
```
* `-G wheel`: sudo 권한을 얻을 수 있는 wheel 그룹에 추가합니다.

### 3. Sudo 권한 부여
```bash
EDITOR=nano visudo
```
파일이 열리면 `%wheel ALL=(ALL:ALL) ALL` 줄을 찾아 앞의 `#` 주석을 제거하고 저장합니다.

---

## 8단계: 부트로더 설치 (GRUB)

시스템이 켜질 때 운영체제를 찾아 실행해 주는 부트로더를 설치합니다.

### 1. GRUB 및 관련 패키지 설치
```bash
pacman -S grub efibootmgr
```
*(듀얼 부팅을 구성하는 경우 `os-prober`, `ntfs-3g` 등을 추가로 설치해야 합니다.)*

### 2. GRUB 시스템에 설치
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

### 3. GRUB 설정 파일 생성
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 9단계: 마무리 및 재부팅

이제 기본적인 설치가 모두 끝났습니다. chroot 환경을 빠져나와 시스템을 재시작합니다.

```bash
exit
umount -R /mnt
reboot
```
*(재부팅 시 설치 USB를 뽑아주세요.)*

컴퓨터가 켜지면 GRUB 부트 메뉴가 나타나고, 이후 로그인 프롬프트에서 앞서 만든 일반 사용자 계정으로 로그인합니다.

---

## 10단계(선택): 데스크톱 환경(GUI) 구축

검은 터미널 창을 벗어나 마우스로 조작 가능한 그래픽 환경(GNOME)을 구축합니다. 영상의 후반부 작업입니다.

### 1. 오디오, 블루투스 등 필수 유틸리티 설치
```bash
sudo pacman -S pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber bluez bluez-utils
sudo systemctl enable bluetooth
```

### 2. 비디오 드라이버 설치 (자신의 그래픽카드에 맞게)
* AMD: `sudo pacman -S xf86-video-amdgpu`
* NVIDIA: `sudo pacman -S nvidia nvidia-utils`
* Intel: `sudo pacman -S xf86-video-intel`

### 3. 데스크톱 환경 (GNOME) 설치
```bash
sudo pacman -S gnome gdm
```
*(선택적으로 `gnome-tweaks` 등 부가 패키지 설치 가능)*

### 4. 디스플레이 매니저(GDM) 활성화
```bash
sudo systemctl enable gdm
```

### 5. 최종 재부팅
```bash
sudo reboot
```
재부팅 후 화려한 그래픽 로그인 화면이 나타나면 완벽하게 Arch Linux 설치에 성공한 것입니다!