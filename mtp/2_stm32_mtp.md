STM32H750VBT6를 사용해 .hwp/.doc 파일만 USB microSD에 저장하는 구현을 위해 다음 단계를 추천합니다. 주요 구성 요소인 USBX(MTP), FatFs, microSD 제어를 통합해야 합니다.

### 1. **하드웨어 설정**  
- **STM32H750VBT6**와 **microSD 카드** 연결:  
  `SDMMC 인터페이스`를 사용해 고속 통신 구현 (SPI 대비 성능 우수).  
  - SDMMC 핀 구성: `CK`, `CMD`, `D0-D3` 연결 필수[2][3].  
- **USB OTG** 설정: `USB_OTG_FS` 또는 `USB_OTG_HS`를 디바이스 모드로 활성화[6][7].

### 2. **소프트웨어 스택 통합**  
#### a) **FatFs + microSD 초기화**  
```c
/* SDMMC 및 FatFs 설정 예시 */
FATFS fs;  // FatFs 객체
FIL fil;    // 파일 객체

// microSD 마운트
f_mount(&fs, "", 0); 
```
- **FAT32 포맷** 필수: Windows 호환성 위해 `f_mkfs()`로 포맷[1][5].  
- SDMMC 클럭: **48MHz 이하**로 제한 (STM32H7 사양 준수)[1].

#### b) **USBX + MTP 설정**  
```c
/* USBX MTP 디바이스 예시 */
ux_device_class_mtp_activate();  // MTP 클래스 활성화
```
- **MTP 필터링 핵심**:  
  파일 전송 시 확장자 검사 로직 추가:
  ```c
  if (is_valid_extension(filename)) { // .hwp/.doc 검사
      f_open(&fil, filename, FA_WRITE); 
      f_write(&fil, data, size, &bw);
  }
  ```
  - `is_valid_extension()` 함수에서 `strcmp()`로 `.hwp`, `.doc` 확인[6].

### 3. **파일 필터링 구현**  
#### MTP 이벤트 핸들러 수정  
```c
/* 파일 수신 시 확장자 체크 로직 추가 */
UINT mtp_data_callback(UX_MTP_EVENT *event) {
    char *ext = strrchr(event->filename, '.'); // 확장자 추출
    if (ext && (strcmp(ext, ".hwp")==0 || strcmp(ext, ".doc")==0)) {
        // SD 카드 저장 로직
        f_open(...); 
        f_write(...);
    } else {
        return UX_ERROR; // 차단
    }
}
```
- **Windows 호환성**: MTP 프로토콜은 파일 필터링을 지원하지 않으므로 **디바이스 측에서 거부**해야 함[7].

### 4. **STM32CubeMX 설정**  
1. **SDMMC**: 4-bit 모드, 클럭 분할기 **≥4** (안정성)[2][3].  
2. **USB_OTG**: 디바이스 모드, MTP 클래스 선택.  
3. **Middleware**:  
   - **FatFs** 활성화 → `USER_DRIVER` 선택.  
   - **USBX** 활성화 → `MTP Device` 추가.  

### 5. **테스트 시나리오**  
1. Windows에서 MTP 디바이스로 STM32 인식 확인.  
2. `.hwp` 파일 전송 → microSD에 저장됨.  
3. `.txt` 파일 전송 → **거부됨** (Error 체크).  

### 문제 해결 팁  
- **SD 카드 인식 실패**:  
  - 클럭 분할기 증가 (`SDMMC_INIT_CLK_DIV=10`으로 시작).  
  - `f_mount()` 실패 시 `f_mkfs()`로 포맷 재시도[5][4].  
- **MTP 연결 안정성**:  
  USB D+/-에 **22Ω 저항** 직렬 연결로 노이즈 감소[7].  

> 이 구현은 **USBX의 MTP 클래스**와 **FatFs의 파일 작업**을 결합하며, 확장자 필터링은 **디바이스 측에서 강제 적용**됩니다. STM32CubeIDE에서 USBX 및 FatFs 미들웨어를 활성화하면 기본 코드 템플릿이 생성됩니다[6][7].

[1] https://blog.naver.com/specialist0/221010885789
[2] https://deepbluembedded.com/stm32-sdmmc-tutorial-examples-dma/
[3] https://deepbluembedded.com/stm32-sdio-sd-card-example-fatfs-tutorial/
[4] https://deepbluembedded.com/stm32-sd-card-spi-fatfs-tutorial-examples/
[5] https://www.youtube.com/watch?v=I9KDN1o6924
[6] https://community.st.com/t5/stm32-mcus-embedded-software/fatfs-using-usb-mass-storage-key/td-p/329513
[7] https://www.youtube.com/watch?v=aEwwQMdKd-c
[8] https://01001000.xyz/2020-08-09-Tutorial-STM32CubeIDE-SD-card/
[9] https://community.st.com/t5/stm32-mcus-embedded-software/stm32l4-spi-sd-card-with-fatfs-and-usb-msc/td-p/131693
[10] https://nexp.tistory.com/m/2967?category=810135
