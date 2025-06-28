## STM32에서 MTP 구현 및 STM32CubeIDE 활용

**STM32CubeIDE 개요**

STM32CubeIDE는 STMicroelectronics에서 공식 제공하는 STM32 시리즈 MCU 전용 통합 개발 환경(IDE)이다. Eclipse/CDT 기반으로, 코드 에디팅, 컴파일, 디버깅, 주변장치 및 미들웨어 설정, 코드 자동 생성 등 임베디드 개발에 필요한 모든 기능을 제공한다. STM32CubeMX의 기능이 통합되어 있어, 프로젝트 생성 시 MCU/보드 선택, 핀맵 및 클럭, USB 등 주변장치 설정, 미들웨어 추가가 GUI로 가능하며, 설정 변경 후 코드 재생성도 지원한다[1][4][5][6][8].

**MTP(Media Transfer Protocol) 지원 현황**

STM32CubeIDE 및 STM32CubeMX에서 기본적으로 제공하는 USB Device Library에는 MTP 클래스가 공식적으로 포함되어 있지 않다. 커뮤니티 포럼에서도 CubeMX(또는 CubeIDE)에서 MTP 클래스를 직접 선택해 생성하는 기능은 제공되지 않는다는 답변이 있다[3]. 즉, STM32CubeMX의 USB 미들웨어 옵션에는 MTP가 없다.

다만, STMicroelectronics의 공식 GitHub 저장소(예: STM32CubeH7)에는 MTP 클래스 소스가 포함되어 있으나, 예제 애플리케이션이나 CubeMX 연동은 제공되지 않는다. 즉, CubeIDE에서 프로젝트를 생성할 때 MTP 클래스를 자동으로 추가하는 기능은 없고, 직접 소스를 추가해 구현해야 한다[3].

**STM32에서 MTP 구현 방법**

- STM32CubeIDE에서 USB Device를 "Custom Class"로 설정해 프로젝트를 생성한다.
- ST의 GitHub(예: STM32CubeH7의 Middlewares/ST/STM32_USB_Device_Library/Class/MTP)에서 MTP 관련 소스를 수동으로 프로젝트에 추가한다.
- USB 엔드포인트, 디스크/파일시스템 연동, MTP 프로토콜 핸들링 등은 직접 구현해야 한다.
- CubeIDE에서 자동 코드 생성 시, 사용자 코드 영역에만 커스텀 코드를 작성해야 재생성 시 코드 손실을 막을 수 있다[7].

**정리**

- STM32CubeIDE는 STM32 개발에 최적화된 공식 IDE로, USB 등 다양한 주변장치 및 미들웨어 설정이 가능하다[1][4][5][6][8].
- MTP 클래스는 CubeIDE/CubeMX에서 기본 제공하지 않으므로, 직접 소스 추가 및 구현이 필요하다[3].
- 공식 GitHub 저장소에서 MTP 소스를 참고할 수 있으나, 통합 예제나 자동화된 CubeMX 지원은 없다[3].
- 프로젝트 생성 및 커스텀 미들웨어 추가, 디버깅, 펌웨어 다운로드 등은 CubeIDE에서 통합적으로 지원된다.

STM32에서 MTP를 구현하려면 CubeIDE로 프로젝트를 생성한 뒤, MTP 소스를 별도로 추가하고, USB 및 파일시스템 연동을 직접 처리해야 한다. CubeIDE 자체는 MTP 미들웨어를 자동으로 지원하지 않는다.

[1] https://www.st.com/en/development-tools/stm32cubeide.html
[2] https://vuzwa.tistory.com/entry/STM32-4-STM32CubeIDE-%EC%82%AC%EC%9A%A9%EB%B0%A9%EB%B2%95
[3] https://community.st.com/t5/stm32cubeide-mcus/mtp-class-in-usb-device-library/td-p/237149
[4] https://spacebike.tistory.com/14
[5] https://raspberrystory.tistory.com/208
[6] https://keepat-it.tistory.com/26
[7] https://louie0724.tistory.com/353
[8] https://vuzwa.tistory.com/entry/STM32-3-STM32-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0STM32CubeIDE-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0
[9] https://www.youtube.com/watch?v=6PXJk3yhfNY
[10] https://blog.naver.com/chandong83/221652968153
