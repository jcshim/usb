좋습니다! 초등학생도 따라할 수 있을 만큼 쉽게, STM32H750 보드와 STM32CubeIDE를 이용해서 **USB MTP 방식으로 특정 문서 파일(.hwp, .doc, .txt 등)만 microSD에 복사할 수 있도록 하는 장치**를 만드는 방법을 아래와 같이 단계별로 설명해드릴게요.

---

## 🧠 먼저, 간단 정리부터!

* **MTP**는 USB를 통해 스마트폰처럼 **파일만 주고받는 방식**이에요.
* 우리는 STM32 보드를 **USB 드라이브처럼 보이게** 만들고,
* 사용자가 `.hwp`, `.doc`, `.txt` 같은 문서만 **microSD에 저장**할 수 있게 만들 거예요.

---

## 🛠 준비물

* STM32H750VBT6 보드
* STM32CubeIDE 최신버전
* USB 케이블
* microSD 카드 + 보드에 연결된 SD카드 슬롯
* Windows PC

---

## 🧭 단계별 따라하기

### 1단계. STM32CubeIDE에서 기본 프로젝트 만들기

1. STM32CubeIDE 실행
2. **\[New STM32 Project] → \[보드 이름: STM32H750VBTx 선택] → Next**
3. Project Name: `USB_MTP_Drive` 로 입력 → Finish

---

### 2단계. USB 장치 기능 설정 (Custom Class)

1. **.ioc 파일 열기**

2. 왼쪽 메뉴에서 **Middleware > USB\_Device** 클릭
   → **Mode: Custom Human Interface Device Class** 선택

3. \[Project Manager] 탭에서 코드 생성 옵션 확인:

   * "Generate under Root" 체크
   * "Generate Peripheral initialization as a pair of '.c/.h' files per peripheral" 선택

4. 오른쪽 상단 **\[Project > Generate Code]** 클릭

---

### 3단계. MTP(Media Transfer Protocol) 기능 넣기

> STM32CubeIDE 기본에는 MTP가 없음 → **ST GitHub에서 수동으로 넣어야 해요**

#### 📦 [GitHub에서 MTP 코드 받기](https://www.youtube.com/watch?v=wOC04mQK9Nw)

1. 웹 브라우저에서 이동:

   * [https://github.com/STMicroelectronics/STM32CubeH7](https://github.com/STMicroelectronics/STM32CubeH7)
2. 경로:

   ```
   Middlewares/ST/STM32_USB_Device_Library/Class/MTP
   ```
3. `MTP` 폴더 전체를 복사해서 CubeIDE의 프로젝트 내 다음 경로에 붙여넣기:

   ```
   USB_MTP_Drive/Middlewares/ST/STM32_USB_Device_Library/Class/MTP
   ```

---

### 4단계. 프로젝트에 MTP 연결하기

1. `usbd_customhid_if.c` 대신 → `usbd_mtp_if.c` 를 사용
2. `usb_device.c`에서 `USBD_CUSTOM_HID_RegisterInterface()` 대신:

   ```c
   extern USBD_ClassTypeDef  USBD_MTP;
   USBD_RegisterClass(&hUsbDeviceFS, &USBD_MTP);
   USBD_MTP_RegisterInterface(&hUsbDeviceFS, &USBD_Interface_fops_FS);
   ```
3. MTP 핸들러 내부에서 다음과 같은 조건을 구현:

   * `.hwp`, `.doc`, `.txt` 파일만 허용
   * 나머지는 무시하거나 오류 전송

---

### 5단계. microSD 연동하기

1. .ioc에서 SDMMC1 또는 SDMMC2 활성화
2. FATFS 미들웨어도 활성화
3. `f_mount`, `f_open`, `f_write` 등을 사용해
   MTP로 받은 파일을 microSD에 저장

```c
if (check_extension(filename, ".txt") || check_extension(filename, ".hwp") || check_extension(filename, ".doc")) {
   f_open(&fil, filename, FA_WRITE | FA_CREATE_ALWAYS);
   f_write(&fil, data, size, &bw);
   f_close(&fil);
}
```

---

### 6단계. USB 연결 후 Windows에서 확인

* 보드를 PC에 USB로 연결하면 스마트폰처럼 **"파일 전송" 장치로 인식됨**
* `.txt`, `.doc`, `.hwp`만 저장됨 (나머지는 무시)

---

## 🎒 보충 설명 (쉽게 이해하기)

| 용어           | 설명                             |
| ------------ | ------------------------------ |
| MTP          | 스마트폰처럼 파일만 주고받는 USB 방식         |
| microSD      | STM32 보드에 꽂는 작은 저장장치           |
| FATFS        | SD카드에서 파일을 읽고 쓰게 해주는 코드        |
| CubeIDE      | STM32 보드용 프로그램을 만드는 툴          |
| Custom Class | 내가 원하는 USB 기능을 만들 수 있도록 해주는 설정 |

---

## 🧩 도움이 되는 팁

* CubeIDE에서 코드를 수정할 땐, **`/* USER CODE BEGIN`** 과 **`END`** 사이에만 코딩하세요!
* 그래야 나중에 "코드 재생성"해도 코드가 사라지지 않아요.

---

## 🔚 마무리

이제 이 프로젝트는 **Windows에서 드라이브처럼 보이고**,
`.txt`, `.doc`, `.hwp` 파일만 microSD에 저장되는 \*\*"문서 전용 USB 저장장치"\*\*가 됩니다!

---

필요하시면 각 단계별 **코드 예제**나 **화면 캡처**, **.ioc 설정파일**도 같이 드릴 수 있어요.
다음 단계로 진행하시겠습니까? (예: “4단계 MTP 연동 코드 좀 보여줘”)
