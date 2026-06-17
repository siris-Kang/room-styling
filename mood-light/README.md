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

ESP32-WROOM-32E 기반 DevKit, BLE, 5V 어댑터 사용

개발 환경은 ESP-IDF 기반

## 4. 사용 예정 하드웨어

### MCU

ESP32-WROOM-32E 기반 DevKit 개발보드

선정 이유

* BLE 지원
* Wi-Fi 확장 가능
* ESP-IDF 지원
* FreeRTOS 기반 개발 가능
* 공식 예제와 문서 기반으로 초기 개발이 쉬움
* USB-UART를 통한 펌웨어 업로드와 시리얼 모니터링이 편리함
* 모듈 단품보다 개발보드 형태가 핀맵 확인, USB 연결, 전원 안정성 측면이 좋음

주의 사항

* ESP32-WROOM-32E는 Xtensa LX6 듀얼 코어 기반 ESP32 모듈임
* GPIO는 3.3V 로직으로 동작, 외부 모듈 제어 신호도 3.3V 입력을 지원해야 함
* LED 스트립 전류는 ESP32-WROOM-32E GPIO 또는 3.3V 핀에서 직접 공급하지 않음
* PWM 출력 핀은 실제 DevKit 보드에 노출된 GPIO 중 LEDC PWM 사용이 가능한 핀으로 선정함
* PWM 후보 핀은 GPIO 16, 17, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33 중 실제 보드에 노출된 핀으로 우선 검토함
* GPIO 34 ~ 39는 입력 전용이므로 PWM 출력용으로 사용하지 않음
* 부팅 스트랩 핀(GPIO 0, 2, 4, 5, 12, 15)은 외부 회로 연결 시 부팅 상태에 영향을 줄 수 있으므로 우선 피함

### LED 구동부

MOSFET 트리거 스위치 PWM 제어 모듈

역할

* ESP32-WROOM-32E GPIO의 3.3V PWM 신호 수신
* LED 스트립 전류 제어
* 밝기 조절 수행
* MCU GPIO가 LED 스트립 전류를 직접 부담하지 않도록 분리

선정 이유

* MOSFET 단품, 게이트 저항, 풀다운 저항을 직접 구성하지 않아도 됨
* VIN, OUT, SIG, GND 단자가 있어 배선이 단순함
* PWM 입력을 지원하므로 LED 밝기 제어에 적합함
* 3.3V High 신호를 지원하는 모듈이므로 ESP32-WROOM-32E GPIO로 제어 가능함

기본 연결

* VIN+ : 5V 어댑터 +
* VIN- : 5V 어댑터 -
* OUT+ : LED 스트립 +
* OUT- : LED 스트립 -
* SIG : ESP32-WROOM-32E PWM GPIO
* GND : ESP32-WROOM-32E GND와 공통 연결

주의 사항

* 모듈에 표시된 160A, 200A 등의 전류 표기는 실제 연속 사용 가능 전류로 보지 않음
* 본 프로젝트에서는 소형 5V LED 스트립 제어 용도로만 사용함
* 장시간 최대 밝기 구동 시 MOSFET 모듈, LED 스트립, 어댑터 발열을 확인함

## 5. 시스템 구조

```text
스마트폰 앱 또는 nRF Connect
    ↓ BLE
ESP32-WROOM-32E DevKit
    ↓ PWM GPIO
MOSFET 트리거 스위치 PWM 제어 모듈
    ↓
5V 웜화이트 COB LED 스트립
```

전원 구조

```text
5V 어댑터
    ├── ESP32-WROOM-32E DevKit
    └── MOSFET 모듈 VIN+ / VIN-
            ↓
        LED 스트립 OUT+ / OUT-

ESP32-WROOM-32E GND와 MOSFET 모듈 GND 공통 연결
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

* 스마트폰 앱과 ESP32-WROOM-32E 연결
* 전원 ON/OFF 명령 수신
* 밝기 값 수신
* 타이머 설정값 수신

초기 테스트 도구

* nRF Connect 앱
* BLE Characteristic 직접 Write 테스트

### 밝기 제어

ESP32-WROOM-32E LEDC PWM 사용

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

1. ESP32-WROOM-32E 개발보드 환경 설정
2. Blink 예제 실행
3. LEDC PWM 예제 실행
4. MOSFET을 이용한 LED 스트립 밝기 제어
5. BLE GATT Server 예제 실행
6. nRF Connect로 밝기 값 전송 테스트
7. ON/OFF 명령 처리
8. 밝기 슬라이더 연동
9. 자동 꺼짐 타이머 추가
10. Android 앱 제작

## 9. 부품 목록

### 필수 부품

* ESP32-WROOM-32E 기반 DevKit 개발보드
* 5V 웜화이트 COB LED 스트립
* MOSFET 트리거 스위치 PWM 제어 모듈
* 5V 2A 이상 어댑터
* 개발보드에 맞는 USB 케이블
* 전선
* 점퍼선
* 브레드보드 또는 만능기판
* 확산 커버 또는 간접광 설치 구조물

### 테스트 및 안정화용 부품

* 멀티미터
* 470uF ~ 1000uF 전해콘덴서
* 택트 스위치
* 10kΩ 저항
* 수축튜브
* 양면테이프 또는 케이블 고정 클립

### LED 구동부 구성 기준

초기 버전에서는 MOSFET 단품을 직접 배선하지 않고, MOSFET 트리거 스위치 PWM 제어 모듈을 사용한다.

해당 모듈은 MOSFET, 부하 연결 단자, PWM 제어 입력 단자를 포함하고 있으므로 LED 스트립 제어 회로를 직접 구성할 필요가 없다.

ESP32-WROOM-32E의 PWM GPIO는 MOSFET 모듈의 SIG 입력에 연결하고, LED 스트립 전류는 MOSFET 모듈을 통해 제어한다.

ESP32-WROOM-32E GPIO 또는 3.3V 핀에서 LED 스트립 전류를 직접 공급하지 않는다.

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

5V 어댑터 기반 전원 공급

배터리 내장 구조는 초기 버전에서 제외

ESP32-WROOM-32E와 LED 스트립의 GND 공통 연결

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

타이머 동작은 앱이 아닌 ESP32-WROOM-32E 내부에서 처리

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
