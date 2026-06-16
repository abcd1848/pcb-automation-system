# AI 기반 PCB 양면 불량 검출 및 자동 분류·AGV 무인 운반 시스템

PCB 양면의 부품 누락, 시리얼 번호 및 뒷면의 긁힘을 머신비전으로 검사하고, 판정 결과에 따라 불량 PCB를 자동 배출하거나 정상 PCB를 박스에 적재한 뒤 AGV 지게차가 창고까지 운반하는 스마트팩토리 자동화 프로젝트입니다.



| 구분 | 내용 |
|---|---|
| 수행 기간 | 2026.05.14 ~ 2026.06.19 |
| 인원 | 5명 |
| 담당 역할 | 로봇팔을 제외한 전체 하드웨어 설계·제작 및 통합 제어 |
| 개발 환경 | Arduino IDE, OpenPLC, C/C++, RS-232, TCP/IP |

[▶ 프로젝트 동작 영상 보기](https://youtu.be/o2dLP7AgSdw?si=CWnDbKM6D3dm-R-M)

---

## 하드웨어 구성

![전체 하드웨어 구성](./docs/images/hardware_overview.png)

---

## 시스템 동작 순서

1. 투입 위치의 로봇팔이 PCB 기판을 메인 컨베이어 위에 올립니다.
2. 메인 컨베이어가 PCB를 카메라 검사 위치까지 이동시킵니다.
3. 머신비전 시스템이 PCB 양면의 부품 누락, 시리얼 번호 및 긁힘을 검사합니다.
4. 불량 판정이 내려지면 전기 실린더가 PCB를 옆으로 밀어 불량함으로 배출합니다.
5. 정상 PCB는 메인 컨베이어 끝까지 이동합니다.
6. 반대편 로봇팔이 정상 PCB를 집어 팔레트 이송형 컨베이어 위의 박스에 적재합니다.
7. 정상 PCB 2개가 적재되면 팔레트 이송형 컨베이어가 박스를 AGV 인수 위치로 이동시킵니다.
8. AGV 지게차가 박스를 들어 올리고 라인을 따라 창고로 이동합니다.
9. 창고의 지정된 위치에 박스를 내려놓은 뒤 초기 위치로 복귀합니다.
10. 이후 같은 공정을 반복합니다.
11. 비상정지 신호가 발생하면 컨베이어, 실린더 및 AGV의 동작을 정지합니다.

---

## PCB 분류 동작

![PCB 분류 흐름](./docs/images/pcb_sorting_flow.png)

---

## 불량 PCB 배출 실린더

![전기 실린더](./docs/images/electric_cylinder.png)

### 실제 배출 동작

![실린더 배출 동작](./docs/images/cylinder_rejection_demo.gif)

### OpenPLC Ladder 프로그램

불량 판정 신호가 들어오면 `Fwd_Pin` 출력이 동작하도록 OpenPLC 기반 Ladder 프로그램을 작성했습니다.

Pin Mapping에서 전진 출력과 후진 출력을 설정했으며, 실제 동작에서는 불량 PCB를 불량함으로 밀어내는 동작에 사용했습니다.

![OpenPLC Ladder 프로그램](./docs/plc/openplc_ladder.png)

---

## 팔레트 이송형 컨베이어

![팔레트 이송형 컨베이어](./docs/images/pallet_conveyor.png)

---

## AGV 지게차

![AGV 지게차](./docs/images/agv_forklift.png)

---

## 회로도

### 메인 제어 회로

Arduino Mega, 모터 드라이버, 스테핑 모터 드라이버, 센서 및 부저를 연결한 회로입니다.

![메인 제어 회로](./docs/circuit/main_controller_circuit.png)

### AGV 제어 회로

Arduino UNO 기반으로 주행 모터, 모터 드라이버, 센서 및 서보모터를 연결한 회로입니다.

![AGV 제어 회로](./docs/circuit/agv_circuit.png)

### 컨베이어 제어 회로

Arduino Uno, 적외선 센서, 모터 드라이버 및 컨베이어 모터를 연결한 회로입니다.

![컨베이어 제어 회로](./docs/circuit/conveyor_circuit.png)

---

## 소스 코드

| 파일 | 내용 |
|---|---|
| [`AGV.ino`](./AGV.ino) | AGV 지게차의 후진, 회전, 전진 및 부저 제어 |
| [`Conveyor.ino`](./Conveyor.ino) | 적외선 센서를 이용한 PCB 감지와 컨베이어 정지·재시작 제어 |
| [`Cylinder.ino`](./Cylinder.ino) | 불량 판정 통신, 전기 실린더 전진·후진, 비상정지 및 상태 전송 |

---

## 내가 맡은 부분

- 로봇팔을 제외한 전체 하드웨어 시스템 구성 및 설계
- PCB 검사 컨베이어와 팔레트 이송형 컨베이어 제작
- Arduino Mega 및 Uno 기반 제어 회로와 전체 배선 구성
- 센서, 모터 드라이버, 부저, 비상정지 버튼 및 전원부 연결
- 적외선 센서를 이용한 PCB 위치 감지 기능 구현
- 불량 PCB 배출을 위한 전기 실린더 구조 및 제어 기능 구현
- 실린더 배출 동작을 위한 OpenPLC Ladder 프로그램 작성
- AGV 지게차의 구동부, 센서, 부저 및 전원부 장착
- AGV의 후진, 회전, 주행, 운반 및 복귀 동작 구현
- PC와 Arduino 사이의 RS-232 통신과 장치 간 신호 연동
- 전체 자동화 공정의 통합 테스트 및 오류 수정
- 회로도 제작과 발표자료 제작

---

## 저장소 구성

```text
pcb-automation-system
├── AGV.ino
├── Conveyor.ino
├── Cylinder.ino
├── README.md
└── docs
    ├── circuit
    │   ├── main_controller_circuit.png
    │   ├── agv_circuit.png
    │   └── conveyor_circuit.png
    ├── images
    │   ├── hardware_overview.png
    │   ├── electric_cylinder.png
    │   ├── cylinder_rejection_demo.gif
    │   ├── pcb_sorting_flow.png
    │   ├── pallet_conveyor.png
    │   └── agv_forklift.png
    └── plc
        └── openplc_ladder.png
```
