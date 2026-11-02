---
layout: single
title: "Hyprland 완벽 구축 상세 가이드"
date: 2026-11-02 13:21:00 +0900
---


# Arch Linux + Hyprland 완벽 구축 상세 가이드

본 가이드는 Josean Martinez의 유튜브 영상 ["How I Setup Hyprland For An Incredible Arch Linux Experience"](https://www.youtube.com/watch?v=PEgDssV0nW0)를 바탕으로 작성된 Hyprland 실무 세팅 매뉴얼입니다. 아무것도 없는 Arch Linux 상태에서 화려하고 효율적인 타일링 윈도우 매니저 환경을 완성하는 전 과정을 다룹니다.

## 1단계: 사전 준비 및 필수 패키지 설치

### 1. 시스템 업데이트 및 yay(AUR 헬퍼) 설치
Arch User Repository(AUR)에서 다양한 커스텀 패키지를 받기 위해 `yay`를 먼저 설치해야 합니다.

```bash
# 시스템 최신화
sudo pacman -Syu

# 기본 개발 도구 및 git 설치
sudo pacman -S base-devel git

# yay 설치
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
```

### 2. Hyprland 및 터미널 설치
기본적인 타일링 윈도우 매니저와 터미널 에뮬레이터(Kitty)를 설치합니다.

```bash
yay -S hyprland kitty
```

### 3. 필수 폰트 설치
시스템 폰트, 이모지, 그리고 개발 및 터미널 환경에서 아이콘이 깨지지 않도록 Nerd Font를 설치합니다.

```bash
yay -S noto-fonts noto-fonts-cjk noto-fonts-emoji ttf-jetbrains-mono-nerd
```

## 2단계: 데스크톱 기반 유틸리티 구성

Hyprland는 순수 윈도우 매니저이므로, 오디오, 네트워크, 알림 등을 처리할 백엔드 유틸리티를 직접 구성해야 합니다.

### 1. 오디오 및 미디어 제어
```bash
yay -S pipewire wireplumber pavucontrol
```

### 2. 알림 데몬 및 인증 에이전트
바탕화면 알림(`mako`)과 GUI 프로그램 실행 시 관리자 권한을 요구하는 팝업(`hyprpolkitagent`)을 설치합니다.
```bash
yay -S mako hyprpolkitagent
```

### 3. XDG 포털 및 Wayland 호환성 패키지
화면 공유나 파일 선택 창이 제대로 뜨도록 포털을 설치합니다.
```bash
yay -S xdg-desktop-portal-hyprland qt5-wayland qt6-wayland
```

## 3단계: Hyprland 환경 설정 (Configuration)

Hyprland의 설정 파일은 `~/.config/hypr/hyprland.conf` (또는 사용자 환경에 따라 분리된 lua/conf 파일)에 위치합니다.

### 1. 모니터 설정 및 배율 (Scaling)
터미널에서 `hyprctl monitors all`을 입력하여 자신의 모니터 이름(예: `DP-1`, `HDMI-A-1`)을 확인한 뒤 설정 파일 상단에 추가합니다.

```conf
# 해상도, 주사율, 위치, 배율 설정
monitor=DP-1,2560x1440@144,0x0,1.25
monitor=,preferred,auto,auto
```

### 2. 외관 (Look & Feel) 설정
창 사이의 간격, 테두리, 투명도 등을 지정합니다.

```conf
general {
    gaps_in = 5
    gaps_out = 10
    border_size = 3
    col.active_border = rgba(33ccffee) rgba(00ff99ee) 45deg
    col.inactive_border = rgba(595959aa)
    layout = dwindle
}

decoration {
    rounding = 10
    active_opacity = 1.0
    inactive_opacity = 0.85
    drop_shadow = true
}
```

### 3. 키보드 입력 감도 향상
빠른 타이핑을 위해 키 반복 속도를 설정합니다.

```conf
input {
    kb_layout = us
    repeat_rate = 25
    repeat_delay = 300
    follow_mouse = 1
}
```

## 4단계: 키 바인딩 (단축키 맵핑)

키보드 중심의 빠른 워크플로우를 위해 Vim 스타일의 단축키를 설정합니다. 기본 모디파이어 키는 `SUPER`(윈도우 키)입니다.

```conf
$mainMod = SUPER
$terminal = kitty
$fileManager = nemo
$browser = chromium

# 프로그램 실행
bind = $mainMod, T, exec, $terminal
bind = $mainMod, B, exec, $browser
bind = $mainMod, F, exec, $fileManager
bind = $mainMod, Q, killactive,
bind = $mainMod SHIFT, Q, exit,

# Vim 스타일 창 포커스 이동 (H, J, K, L)
bind = $mainMod, H, movefocus, l
bind = $mainMod, L, movefocus, r
bind = $mainMod, K, movefocus, u
bind = $mainMod, J, movefocus, d

# 워크스페이스 이동
bind = $mainMod, 1, workspace, 1
bind = $mainMod, 2, workspace, 2
# ... (9까지 반복)

# 창을 특정 워크스페이스로 이동
bind = $mainMod SHIFT, 1, movetoworkspace, 1
bind = $mainMod SHIFT, 2, movetoworkspace, 2
```

## 5단계: 앱 런처 및 다크 테마 통합

### 1. Rofi (앱 런처) 설치 및 설정
Wayland용 Rofi를 설치하고 테마를 적용합니다.

```bash
yay -S rofi-wayland
```
설정 파일에 Rofi 단축키를 추가합니다.
```conf
bind = $mainMod, SPACE, exec, rofi -show drun
bind = $mainMod SHIFT, SPACE, exec, rofi -show run
```

### 2. 전역 다크 모드 및 GTK 테마 (NWG-Look)
모든 창을 다크 모드로 통일하고 예쁜 아이콘을 적용하기 위한 설정입니다.

```bash
yay -S nwg-look gnome-themes-extra papirus-icon-theme
```
설치 후 터미널에서 `nwg-look`을 실행하여 다크 테마(Adwaita-dark 등)와 아이콘(Papirus-Dark)을 선택하고 적용(Apply)합니다.

## 6단계: 상태 표시줄(Waybar) 및 배경화면

### 1. 배경화면 설정 (Hyprpaper)
```bash
yay -S hyprpaper
```
`~/.config/hypr/hyprpaper.conf` 파일을 생성하고 이미지를 지정합니다.
```conf
preload = /경로/배경화면.jpg
wallpaper = DP-1,/경로/배경화면.jpg
```

### 2. Waybar 연동
상태 표시줄인 Waybar를 설치하고 자동 실행에 추가합니다.
```bash
yay -S waybar
```

### 3. 자동 실행 (Autostart) 설정
Hyprland가 켜질 때 백그라운드 앱들이 함께 켜지도록 `hyprland.conf` 맨 위에 추가합니다.

```conf
exec-once = waybar &
exec-once = hyprpaper &
exec-once = mako &
exec-once = systemctl --user start hyprpolkitagent
```

## 마무리
모든 설정을 마치고 `SUPER + SHIFT + Q`를 눌러 Hyprland를 종료한 뒤 재진입하거나, 컴퓨터를 재부팅(`reboot`)하면 완벽하게 커스텀된 나만의 타일링 데스크톱 환경을 만날 수 있습니다!