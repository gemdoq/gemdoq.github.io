---
layout: single
title: "Windows 10에서 후면 스피커와 전면 헤드셋 독립적으로 분리하기"
date: 2025-08-25 13:58:38 +0900
categories: windows
tags: [Realtek, Audio, Windows10]
typora-root-url: ../
---

# ✈️ Windows 10에서 후면 스피커와 전면 헤드셋 분리하기

## 🎯 개요

Windows 10에서 YouTube Music은 후면 스피커로, League of Legends는 전면 헤드셋으로 오디오를 따로 출력하고자 함
Realtek Audio Console 설치에 실패했지만, `RtkNGUI64.exe`로 Realtek HD Audio Manager를 실행해 다중 스트리밍 설정 과정을 기록  

- 목표: YouTube Music은 스피커로, League of Legends는 헤드셋으로 출력  
- 핵심: Realtek Audio Console 실패 후 `RtkNGUI64.exe`로 다중 스트리밍 활성화  
- 결과: 앱별 오디오를 원하는 장치로 제어  

## 📋 목록

- [ ] 사운드 출력 장치란?  
- [ ] Realtek Audio Console 설치 실패  
- [ ] RtkNGUI64.exe로 다중 스트리밍 설정  
- [ ] Windows에서 앱별 출력 설정  
- [ ] 요약 및 팁  

---

## ✏️ 사운드 출력 장치란?

- 컴퓨터에서 오디오를 출력하는 하드웨어  
- 스피커, 헤드셋 등이 포함  
- 스마트폰에서 음악을 스피커로 듣거나 이어폰으로 듣는 것처럼, 앱이 오디오를 원하는 장치로 보내는 구조  
- Windows는 기본적으로 모든 오디오를 하나의 출력 장치로 보냄  
- Realtek 드라이버로 두 장치를 독립적으로 설정  

---

## ✏️ Realtek Audio Console 설치 실패

최신 Realtek Audio Console로 설정하려 했지만 설치에 계속 실패  

- 문제 상황:  
  - Microsoft Store에서 "Realtek Audio Console" 검색 시 앱 미표시  
  - 공식 링크(`ms-windows-store://pdp/?ProductId=9p2b8mcsvpln`)로도 설치 불가  
  - Store 캐시 초기화(`wsreset.exe`)와 Store 업데이트 후에도 동일  
  - 다른 사용자 후기에서도 설치 실패 사례 다수  
- 원인: DCH 드라이버 전용 앱, Store 서버나 지역 문제 가능성  
- 결론: Realtek Audio Console 포기, `RtkNGUI64.exe`로 전환  

> 💡 왜 중요?  
> Realtek Audio Console 설치 실패는 흔한 문제, 대안으로 Realtek HD Audio Manager 사용  

---

## ✏️ RtkNGUI64.exe로 다중 스트리밍 설정

Realtek Audio Console이 안 되니 `RtkNGUI64.exe`로 Realtek HD Audio Manager 실행  
후면 스피커와 전면 헤드셋을 독립적인 출력 장치로 설정  

### 설정 방법

1. 프로그램 실행:  
   - `C:\Program Files\Realtek\Audio\HDA`에서 `RtkNGUI64.exe` 실행  
   - 또는 제어판 > 하드웨어 및 사운드 > Realtek HD 오디오 관리자  
2. 다중 스트리밍 활성화:  
   - 톱니바퀴(설정) 아이콘 클릭  
   - "아날로그 후면 패널과 전면 패널 출력 장치가 서로 다른 오디오 스트림을 동시에 재생하도록 설정" 체크  
   - "모든 입력 잭을 독립적인 입력 장치로 분리"도 체크  
3. 적용 및 재부팅:  
   - 설정 저장 후 재부팅  
   - Windows 사운드 설정에서 "스피커(Realtek Audio)"와 "헤드폰(Realtek Audio)" 확인  

### 비유로 이해
- 스마트폰에서 음악은 스피커로, 게임 소리는 이어폰으로 듣는 상황  
- `RtkNGUI64.exe` 실행으로 두 출력 경로를 독립적으로 활성화  

> 💡 팁: "동시 재생"이나 "독립 출력" 키워드 확인  

---

## ✏️ Windows에서 앱별 출력 설정

스피커와 헤드셋이 분리되었으니 앱별로 오디오 지정  

1. 사운드 설정 열기:  
   - Windows 키 + I > 시스템 > 사운드  
   - 고급 사운드 옵션 > 앱 볼륨 및 장치 기본 설정  
2. 앱별 출력 지정:  
   - YouTube Music은 "스피커(Realtek Audio)" (후면 스피커)로 설정  
   - League of Legends는 "헤드폰(Realtek Audio)" (전면 헤드셋)으로 설정  
   - 볼륨 조절 가능  
3. 테스트:  
   - YouTube Music과 League of Legends 실행 후 오디오 출력 확인  

> 💡 팁: 앱이 목록에 없으면 실행 후 다시 확인  

---

## ✏️ 개념 비교

| 개념                     | 역할                       | 특징                                                      |
|--------------------------|----------------------------|----------------------------------------------------------|
| 사운드 출력 장치         | 오디오 출력 하드웨어       | - 스마트폰 스피커, 이어폰처럼 오디오 출력 <br> - 스피커, 헤드셋 포함 |
| Realtek HD Audio Manager | 다중 스트리밍 설정 도구    | - `RtkNGUI64.exe`로 실행 <br> - 앱별 오디오 라우팅 지원   |
| 앱별 출력 설정           | 앱마다 출력 장치 지정      | - Windows 설정으로 앱별 출력 제어                         |

---

## 💡 요약

Windows 10에서 후면 스피커와 전면 헤드셋을 독립적으로 분리하려면 Realtek Audio Console 설치 대신 `RtkNGUI64.exe`로 Realtek HD Audio Manager 실행  
다중 스트리밍 활성화 후 Windows 설정에서 앱별 출력 지정  