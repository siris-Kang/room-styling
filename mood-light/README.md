# Mood Light

은은한 간접광 형태의 침실 무드등 제작 프로젝트

## 1. 프로젝트 배경

잠들기 전 방 불을 모두 끈 상태에서 스마트폰을 사용하는 습관이 있음

어둠 속에서 밝은 화면을 오래 볼 경우 화면과 주변 밝기 차이는 눈에 문제를 일으킴

침대에 누운 상태에서 부담 없이 사용할 수 있는 은은한 간접 조명 필요

## 2. 프로젝트 목적

침대 머리 위쪽에 설치하는 간접 무드등 제작

스마트폰 앱을 통한 전원 제어

밝기 조절이 가능한 웜화이트 조명 구현

눈부심을 줄이는 간접광 구조 설계

수면 전 30분 사용을 고려한 낮은 밝기 조명 구현

## 3. 핵심 설계 방향

직접광이 아닌 벽 반사 기반 간접광 사용

차가운 흰빛이 아닌 따뜻한 웜화이트 조명 사용

ESP32-C3, BLE, USB-C 어댑터 사용

개발 환경은 ESP-IDF 기반

## 4. 사용 예정 하드웨어

### MCU

ESP32-C3 개발보드

선정 이유
- BLE 지원, Wi-Fi 확장 가능
- ESP-IDF 지원
- FreeRTOS 기반 개발 가능

### LED

5V 웜화이트 COB LED 스트립

선정 이유
- 부드러운 확산광 구현
- 침실 무드등에 적합한 은은한 빛
- RGB LED 대비 목적에 맞는 단순한 구조
- PWM 밝기 조절 가능

권장 색온도
- 2200K ~ 2700K
- 전구색 계열
- 눈부심과 수면 방해감 감소 목적

### 전원

USB-C 5V 어댑터

선정 이유
- 안정적인 전원 공급
- 배터리 충전 회로 불필요
- 보호회로 설계 부담 감소
- 침대 주변 사용 시 안전성 확보 용이

### LED 구동부

N채널 로직레벨 MOSFET

역할
- ESP32-C3 GPIO의 PWM 신호 수신
- LED 스트립 전류 제어
- 밝기 조절 수행

기본 구성
- MOSFET
- 게이트 직렬 저항
- 게이트 풀다운 저항
- 공통 GND 구성

## 5. 시스템 구조

```text
스마트폰 앱
    ↓ BLE
ESP32-C3
    ↓ PWM
MOSFET
    ↓
웜화이트 COB LED 스트립
```

전원 구조
```
USB-C 5V 전원
    ├── ESP32-C3
    └── LED 스트립

ESP32-C3 GND와 LED GND 공통 연결
```

## 6. 사용 예정 기술

### 펌웨어

* ESP-IDF
* FreeRTOS
* BLE GATT Server
* LEDC PWM Driver
* GPIO Driver
* NVS 저장소

### 통신

BLE 사용

사용 목적

* 스마트폰 앱과 ESP32-C3 연결
* 전원 ON/OFF 명령 수신
* 밝기 값 수신
* 타이머 설정값 수신

초기 테스트 도구

* nRF Connect 앱
* BLE Characteristic 직접 Write 테스트

### 밝기 제어

ESP32-C3 LEDC PWM 사용

제어 방식

* PWM duty 변경
* 0 ~ 100% 밝기 조절
* 최대 밝기 제한

### 상태 저장

NVS 사용 예정

저장 항목

* 마지막 밝기 값
* 마지막 전원 상태
* 기본 타이머 설정값

## 7. 개발 환경

### Firmware

* ESP-IDF
* VS Code
* ESP-IDF Extension
* C language
* FreeRTOS

### Mobile

* Android
* Kotlin
* BLE GATT Client

### Test

* nRF Connect
* Serial Monitor
* ESP-IDF Monitor

## 8. 초기 개발 순서

1. ESP32-C3 개발보드 환경 설정
2. Blink 예제 실행
3. LEDC PWM 예제 실행
4. MOSFET을 이용한 LED 스트립 밝기 제어
5. BLE GATT Server 예제 실행
6. nRF Connect로 밝기 값 전송 테스트
7. ON/OFF 명령 처리
8. 밝기 슬라이더 연동
9. 자동 꺼짐 타이머 추가
10. Android 앱 제작

## 9. 예상 부품 목록

* ESP32-C3 개발보드
* 5V 웜화이트 COB LED 스트립
* N채널 로직레벨 MOSFET
* 220Ω 게이트 직렬 저항
* 100kΩ 게이트 풀다운 저항
* USB-C 5V 어댑터
* 전선
* 브레드보드 또는 만능기판
* 확산 커버 또는 간접광 설치 구조물

## 10. 안전성 및 신뢰성

## 안전 및 신뢰성 설계

### 기본 설계 원칙

침대 주변에서 사용하는 조명 장치이므로 오동작 시 LED를 끄는 방향으로 설계

부팅 직후 기본 상태 OFF

전원 재인가 시 기본 상태 OFF

비정상 입력 수신 시 기존 상태 유지 또는 OFF 처리

최대 밝기 제한 적용

자동 꺼짐 타이머 적용

### 전원 안정성

USB-C 5V 어댑터 기반 전원 공급

배터리 내장 구조는 초기 버전에서 제외

ESP32-C3와 LED 스트립의 GND 공통 연결

LED 전류는 GPIO가 아닌 MOSFET을 통해 제어

LED 소비 전류보다 여유 있는 전원 용량 확보

### LED 제어 안정성

PWM duty 범위 제한

0 ~ 100% 입력값 검증

소프트웨어 최대 밝기 제한

급격한 밝기 변화 방지를 위한 fade in/out 적용 검토

MOSFET 발열 확인

장시간 구동 테스트 수행

### 펌웨어 신뢰성

Watchdog Timer 적용 검토

Brownout 발생 시 안전 상태 복귀

BLE 명령값 범위 검증

잘못된 BLE Write 요청 무시

타이머 동작은 앱이 아닌 ESP32-C3 내부에서 처리

BLE 연결이 끊겨도 자동 꺼짐 동작 유지

### 상태 저장 정책

NVS에 마지막 밝기 값 저장

NVS write 빈도 제한

슬라이더 이동 중 연속 저장 방지

전원 상태는 저장하지 않고 재부팅 시 OFF 처리

### 테스트 항목

부팅 직후 OFF 확인

전원 재인가 후 OFF 확인

BLE 연결 및 해제 테스트

잘못된 밝기 값 수신 테스트

자동 꺼짐 타이머 테스트

최대 밝기 장시간 구동 테스트

MOSFET 및 LED 발열 테스트

케이블 고정 상태 확인
