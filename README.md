# PCB Automation System

컨베이어, 실린더, 로봇팔 및 AGV를 연동하여 PCB 검사와 물류 이송 과정을 자동화한 프로젝트입니다.

## System Overview

PCB 검사 결과에 따라 실린더가 불량품을 분류하고, 정상 PCB는 컨베이어를 통해 적재됩니다. 적재가 완료되면 AGV가 제품을 지정된 위치로 운반합니다.

## System Circuit Diagrams

### 1. Main Controller Circuit

Arduino Mega를 중심으로 DC 모터 드라이버, 스테핑 모터 드라이버, 센서 및 부저를 연결한 회로입니다.

![Main Controller Circuit](./docs/circuit/KakaoTalk_20260616_111030203.png)

### 2. AGV Circuit

Arduino UNO R4 WiFi를 중심으로 주행 모터, 모터 드라이버, 초음파 센서 및 서보모터를 연결한 AGV 제어 회로입니다.

![AGV Circuit](./docs/circuit/KakaoTalk_20260616_111030203_01.png)

### 3. Conveyor Circuit

Arduino Uno와 적외선 센서, DC 모터 드라이버를 이용하여 PCB 감지 및 컨베이어 모터를 제어하는 회로입니다.

![Conveyor Circuit](./docs/circuit/KakaoTalk_20260616_111030203_02.png)

## Source Code

| File | Description |
|---|---|
| `AGV.ino` | AGV 주행 및 라인 트레이싱 제어 |
| `Cylinder.ino` | 불량 PCB 배출 실린더 제어 |
| `Conveyor.ino` | 컨베이어 벨트 및 센서 제어 |

## Main Components

- Arduino Uno
- DC Motor Driver
- Pneumatic Cylinder
- Conveyor Belt
- Line Tracking Sensor
- Servo Motor
- AGV Platform
