# dwm1001-keil-examples

*Note 아래 예제는 DWM1001C의 UWB 특징을 이용한 아주 간단한 어플리케이션으로 되어 있다. 이 예제는 산업용 어플리케이션 용도가 아니고 일반적인 요구사항에 무응하지 못할 수 있다.*

*DWM1001C용 고급 펌웨어는 규칙을 준수하며 https://www.decawave.com/product/dwm1001-module/ 에서 찾을 수 있다.*

## 개요

DWM1001 하드웨어용 C 예제를 포함하고 있어서 DWM1001-DEV 보드에서 사용 가능하다.

DWM1001 모듈은 nrF52832 Soc와 DecaWave의 DW1000 IC 기반의 UWB 및 블루투스 하드웨어이다. 수 천개의 tags까지 가능한 확장 Two_Way-Ranging(TWR) RTLS를 확장할 수 있다.

DWM1001-DEV는 DWM1001 모듈용 개발 보드이다. 개발과 디버깅을 위해서 Jlink를 통합되어 제공하고 있다.
DWM1001관련 더 상세한 정보는 www.decawave.com을 방문하자.

C simple 예제는 DWM1001과 UWB가 제공하는 핵심 기능을 경험할 수 있다. 이 예제들은 DWM1001-DEV를 커스텀마이즈하며 다른 DWM1001 기반 HW에 포팅하기 위해서는 일부 수정이 필요하다.(특히 LED와 버튼 인터페이스)

프로젝트는 다음과 같이 되어 있다: 
```
dwm1001-keil-examples/
├── boards            // DWM1001-DEV 보드 스펙 정의
├── deca_driver       // DW1000 API SW 패키지 2.04 
├── examples          // C simple examples 
│   ├── ss_twr_init   // Single Sided Two Way Ranging Initiator example
│   ├── ss_twr_resp   // Single Sided Two Way Ranging Responder example
│   └── twi_accel     // 2 Wire 인터페이스를 가진 LIS2DH12 accelerometer example
├── nRF5_SDK_14.2.0   // Nordic Semiconductor SDK 14.2 for nrF52832
└── README.md
```
nrF52832 및 nrF SDK에 관한 추가 정보는 http://infocenter.nordicsemi.com/ 를 참고하자.

## 지원하는 IDE

예제들은 다음 IDE를 사용한다 :
* Segger Embedded Studio (SES)
* Keil KEIL µVision

## SES(Segger Embedded Studio)

각 예제는 SES용 emproject proejct 파일을 포함하고 있다. 예제들은 컴파일되어 DWM1001로 로드된다.
프로젝트는 SES version V3.34a로 생성되었다.

SES는 nrF52832 개발에 자유롭게 사용할 수 있다. 결과적으로 이 IDE는 DWM1001 개발에 제약없이 사용할 수 있다.

Segger Embedded Studio에 대한 상세 정보는 https://www.segger.com/products/development-tools/embedded-studio/ 를 참고하자.

nrF52832에 대한 공짜 라이센스에 대한 정보는 https://www.nordicsemi.com/News/News-releases/Product-Related-News/Nordic-Semiconductor-adds-Embedded-Studio-IDE-support-for-nRF51-and-nRF52-SoC-development 를 참고하자.

### SES : 추가 Package

SES IDE를 사용하는 경우 아래 package를 설치하도록 하자. :

Package                                                                                                                           
CMSIS 5 CMSIS-CORE Support Package (version 5.02)                                                                           
CMSIS-CORE Support Package (version 4.05)                                                                           
Nordic Semiconductor nRF CPU Support Package (version 1.06)                                                                           

tools 메뉴에서 package manager를 통해서 SES 자체에서 설치할 수 있다.

## KEIL µVision IDE

각 예제는 Keil µVision IDE를 위한 µVision5 project 파일을 포함하고 있다. 이 예제들은 컴파일 및 DWM1001로 로드된다.
이 프로젝트는 KEIL uVision version V5.24.2.0로 생성되었다. Keil µVision 관련해서 더 많은 정보는 http://www2.keil.com/mdk5/uvision/ 를 참고하세요.

### µVision Error: Flash Download failed - "Cortex-M4"

이 에러는 로드되는 binary와 타겟 HW상에 있는 현재 firmware 사이에 메모리 충돌이 있는 경우에 나타난다. 이 이슈는 타겟 장치의 flash 메모리를 완전히 삭제하는 경우 쉽게 해결할 수 있다. Keil µVision는 완전히 삭제를 수행할 수 없어서 아래와 같은 도구를 사용해야 한다:

* J-flash lite 
* nrfjprog command line script

이슈에 관련된 더 상세한 정보는 다음을 보자. :

https://devzone.nordicsemi.com/f/nordic-q-a/18278/error-flash-download-failed---cortex---m4-while-flashing-softdevice-from-keil-uvision-5

## Example Details 

SS-TWR scheme은 DWM1001 모듈을 2개를 사용해서 구현할 수 있다. 한 개는 initiator로 프로그램하고 다른 한 쪽은 responder로 프로그램한다.

### Single Sided Two Way Ranging -- Initiator

이 예제는 initiator에 대한 소스 코드를 포함하고 있다. initiator는 frame을 보내고 receiver로부터 response를 기다리고 거리는 계산하고 출력을 UART로 보낸다.(serial 터미널에서 나오는 값을 보면 된다.)

```
dwm1001-keil-examples/examples/ss_twr_init/
├── config                    // nrF SDK 14.2 커스텀을 위한 sdk_config.h 파일 포함
├── main.c                    // Initialization 와 main program
├── ss_init_main.c            // Single sided initiator core program
├── UART                      // Uart 
├── SES
│   └── ss_twr_init.emProject // Segger Embedded Studio project
└── Keil uvision
     └── ss_twr_init.uvprojx  // Keil uvision project

```
이 어플리케이션 기능은 main.c와 ss_init_main.c 파일에서 상세한 내용을 볼 수 있다.

정확한 측정값을 가지기 위해서 캘리브레이션이 필요하다. 하드웨어에 따라서 안테나 지연을 조정하는 방식으로 할 수 있다.

### Single Sided Two Way Ranging -- Responder

이 예제는 responder에 대해서 소스 코드를 포함하고 있다. receiver는 initiator로부터 하나의 frame을 수신하고 관련 답변을 전송한다.

```
dwm1001-keil-examples/examples/ss_twr_resp/
├── config                    // nrF SDK 14.2 커스텀을 위한 sdk_config.h 파일 포함
├── main.c                    // Initialization와 main program
├── ss_resp_main.c            // Single sided responder core program
├── SES
│   └── ss_twr_resp.emProject // Segger Embedded Studio project
└── Keil uvision
     └── ss_twr_resp.uvprojx  // Keil uvision project
```
main.c와 ss_resp_main.c에서 상세한 내용을 볼 수 있다.

정확한 측정값을 얻기 위해서 칼리브레이션이 필요하다. 하드웨어에 따라서 안테나 지연을 조정하는 방식으로 할 수 있다.

### Two Wire Interface Accelerometer

이 예제는 LIS2DH12와 nrF52832 사이에 TWI를 구현한다.
blue led는 LIS2DH12가 움직임을 포착하면 빛이 난다. UART로 가속 상태를 리포팅한다.

```
dwm1001-keil-examples/examples/twi_accel/
├── config                    // Contains sdk_config.h file for nrF SDK 14.2 customization
├── LIS2DH12                  // LIS2DH12 (accelerometer) low level driver and api
├── main.c                    // Initialization and main program
├── TWI                       // TWI
├── UART                      // Uart
├── SES
│   └── twi_accel.emProject // Segger Embedded Studio project
└── Keil uvision
     └── twi_accel.uvprojx  // Keil uvision project
```
main.c 파일에 상세한 내용을 확인할 수 있다.




