---
layout: single
title: "Hyprland 미니멀 Waybar 완벽 구축 상세 가이드"
date: 2026-11-02 13:34:00 +0900
---

# Hyprland 미니멀 Waybar 완벽 구축 상세 가이드

본 가이드는 Josean Martinez의 유튜브 영상 ["How To Build The Perfect Minimal Status Bar (hyprland + waybar)"](https://www.youtube.com/watch?v=T5Itsza4PhE)를 바탕으로 작성되었습니다. 복잡하고 지저분한 상태 표시줄 대신, 바탕화면에 떠 있는 듯한(Floating) 깔끔하고 직관적인 미니멀 Waybar를 구축하는 전 과정을 다룹니다.

## 1단계: 필수 패키지 설치 및 기본 디렉토리 구성

### 1. Waybar 및 폰트 설치
상태 표시줄 역할을 할 `waybar`와 아이콘이 깨지지 않게 표시해 줄 `Nerd Font`를 설치합니다. (설정 편집 중 실시간 새로고침을 위한 `inotify-tools`도 함께 설치합니다.)

```bash
yay -S waybar ttf-jetbrains-mono-nerd inotify-tools
```

### 2. 설정 디렉토리 생성
Waybar 설정이 위치할 폴더를 생성하고, 기본 뼈대가 될 파일들을 만듭니다.

```bash
mkdir -p ~/.config/waybar
touch ~/.config/waybar/config.jsonc
touch ~/.config/waybar/style.css
```

## 2단계: 작업 효율을 위한 실시간 새로고침 (Live Reload)

설정이나 CSS를 수정할 때마다 매번 Waybar를 껐다 켜는 것은 매우 번거롭습니다. 파일이 수정되면 자동으로 Waybar를 재시작해 주는 스크립트를 작성합니다.

1. 스크립트 파일 생성:
```bash
nano ~/.config/waybar/launch.sh
```

2. 아래 스크립트를 붙여넣고 저장합니다:

```bash
#!/bin/bash
# 기존 실행 중인 waybar 종료
killall waybar

# 백그라운드에서 waybar 실행
waybar &

# 설정 파일이 변경되면 자동으로 waybar 재시작
while inotifywait -e create,modify ~/.config/waybar/config.jsonc ~/.config/waybar/style.css; do
    killall waybar
    waybar &
done
```

3. 스크립트에 실행 권한을 부여합니다:
```bash
chmod +x ~/.config/waybar/launch.sh
```
*이제 설정 파일을 편집할 때는 터미널에서 `~/.config/waybar/launch.sh`를 실행해 둔 상태로 작업하면 실시간으로 변경 사항을 확인할 수 있습니다.*

## 3단계: 구조 설계 (`config.jsonc`)

`config.jsonc`는 Waybar에 '어떤 정보(모듈)를 표시할지'와 '배치(좌/중/우)'를 결정합니다.

```jsonc
{
    "layer": "top",
    "position": "top",
    "margin-top": 10,
    "margin-left": 10,
    "margin-right": 10,
    "height": 40,
    "spacing": 10,

    // 좌/중/우 모듈 배치
    "modules-left": ["hyprland/workspaces"],
    "modules-center": ["clock"],
    "modules-right": ["tray", "cpu", "memory", "pulseaudio", "battery"],

    // 모듈 상세 설정
    "hyprland/workspaces": {
        "disable-scroll": true,
        "all-outputs": true,
        "format": "{icon}",
        "format-icons": {
            "1": "",
            "2": "",
            "3": "",
            "default": "",
            "active": ""
        }
    },
    
    "clock": {
        "format": "{:%I:%M %p}", // 예: 02:30 PM
        "tooltip-format": "<big>{:%Y %B}</big>\n<tt><small>{calendar}</small></tt>"
    },
    
    "cpu": {
        "format": " {usage}%",
        "tooltip": false,
        "on-click": "kitty -e btop" // 클릭 시 btop 실행
    },
    
    "memory": {
        "format": " {}%"
    },
    
    "pulseaudio": {
        "format": "{icon} {volume}%",
        "format-muted": " Muted",
        "format-icons": {
            "default": ["", "", ""]
        },
        "on-click": "pavucontrol" // 클릭 시 볼륨 제어기 실행
    },
    
    "battery": {
        "states": {
            "warning": 30,
            "critical": 15
        },
        "format": "{icon} {capacity}%",
        "format-icons": ["", "", "", "", ""]
    },
    
    "tray": {
        "icon-size": 18,
        "spacing": 10
    }
}
```
*(주의: 아이콘이 깨져 보인다면 Nerd Font가 제대로 적용되지 않은 것이므로 폰트 설정을 확인해야 합니다.)*

## 4단계: 미니멀 디자인 입히기 (`style.css`)

`style.css`는 둥근 모서리, 투명한 배경, 여백 등 미니멀한 시각적 요소를 담당합니다.

```css
* {
    font-family: "JetBrainsMono Nerd Font", sans-serif;
    font-size: 14px;
    font-weight: bold;
}

/* 전체 바탕을 투명하게 (플로팅 효과) */
window#waybar {
    background-color: transparent;
    color: #ffffff;
}

/* 각 모듈의 공통 디자인 (둥근 모서리와 불투명한 배경) */
#workspaces, #clock, #cpu, #memory, #pulseaudio, #battery, #tray {
    background-color: rgba(30, 30, 46, 0.85); /* 다크 블루 톤 투명도 */
    border-radius: 20px; /* 둥근 모서리 */
    padding: 0 15px;
    margin: 0 5px;
}

/* 워크스페이스(화면) 아이콘 호버 및 활성 상태 색상 */
#workspaces button {
    color: #cdd6f4;
    padding: 0 5px;
}

#workspaces button:hover {
    box-shadow: inherit;
    text-shadow: inherit;
    color: #f38ba8;
}

#workspaces button.active {
    color: #89b4fa; /* 활성화된 화면은 파란색으로 표시 */
}

/* 특정 모듈 색상 포인트 */
#pulseaudio {
    color: #a6e3a1; /* 초록색 */
}

#battery {
    color: #f9e2af; /* 노란색 */
}
```

## 5단계: 부가 유틸리티 연동 (클릭 액션)

Waybar의 모듈을 클릭했을 때 작동할 유틸리티 패키지들을 설치합니다. (JSONC의 `on-click` 속성과 연동됨)

```bash
# btop (터미널 기반 리소스 모니터) 및 pavucontrol (GUI 오디오 제어기)
yay -S btop pavucontrol
```
이제 Waybar에서 CPU나 메모리 영역을 클릭하면 `btop`이 열리고, 오디오 아이콘을 클릭하면 `pavucontrol`이 열리며 상호작용이 가능해집니다.

## 6단계: Hyprland 환경과 완벽 통합 (Autostart)

마지막으로 Hyprland가 시작될 때 방금 만든 Waybar가 자동으로 실행되도록 설정합니다.

1. Hyprland 설정 파일 열기:
```bash
nano ~/.config/hypr/hyprland.conf
```

2. `exec-once` 항목에 아래 내용을 추가합니다:
```bash
exec-once = waybar
```
*(만약 커스텀 스크립트를 등록했다면 `exec-once = ~/.config/waybar/launch.sh &` 로 지정할 수도 있습니다.)*

## 마무리

이제 키보드의 `SUPER + SHIFT + Q` (또는 지정한 종료 단축키)로 Hyprland를 재시작해 보세요. 바탕화면 상단에 둥근 모서리를 가진 투명하고 미니멀한 Waybar가 아름답게 나타날 것입니다! 모듈에 마우스를 올리거나 클릭하며 상호작용을 테스트해 보세요.
