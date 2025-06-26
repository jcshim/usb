USB MSC(Mass Storage Class) 방식으로 **파일 확장자 필터링 구현이 불가능하다는 주장**은 부분적으로 맞습니다. 검색 결과와 기술적 근거를 바탕으로 설명드리겠습니다.

### 🔍 **USB MSC의 기본 동작과 한계**
1. **블록 기반 접근**  
   - USB MSC는 **저수준 섹터(블록) 단위 접근**만 제공합니다[2][3].  
   - 호스트(PC)는 장치(STM32)에게 "LBA(Logical Block Address) X의 512바이트를 읽어줘" 같은 명령만 전달합니다.  
   - 파일 시스템(FAT, NTFS 등) 해석은 **호스트 OS가 전담**합니다[3][4].  

2. **필터링 불가능한 구조**  
   - STM32는 파일 이름, 확장자, 디렉토리 구조를 인식하지 못합니다.  
   - 예시:  
     - PC에서 `COPY *.hwp` 명령을 보내도, STM32는 **"블록 0x1000~0x2000 읽어줘"** 같은 요청만 받습니다.  
     - `.hwp` 파일이 SD 카드의 어떤 블록에 위치하는지 STM32가 알 수 없습니다.

> ***"USB MSC는 호스트가 요청한 섹터를 그대로 반환할 뿐, 파일 단위 필터링은 프로토콜 수준에서 불가능합니다."*** [2][3]

### ⚙️ **구현 가능한 대안: STM32 호스트 모드**
USB MSC로는 불가능하지만, **STM32를 USB 호스트(Host)로 설정**하면 해결됩니다. 이때 필요한 기술 요소는 다음과 같습니다.

#### 1. **필수 컴포넌트**
   | 기술                  | 역할                                                                 |
   |-----------------------|----------------------------------------------------------------------|
   | **USB OTG Host**      | USB 메모리를 직접 제어 (호스트 모드)[3]                              |
   | **FATFS 미들웨어**    | SD 카드/USB 메모리의 파일 시스템 파싱[4]                              |
   | **파일 필터링 로직**  | 확장자(`.hwp`, `.doc`) 기반 파일 선택 후 복사                          |

#### 2. **동작 원리**
   ```mermaid
   graph TD
     A[PC] -- 파일 복사 요청 --> B(STM32)
     B -- 1. SD 카드 마운트 --> C[FATFS]
     C -- 2. 파일 목록 스캔 --> D{확장자 필터링?}
     D -- .hwp/.doc --> E[USB 메모리 복사]
     D -- 기타 파일 --> F[무시]
   ```

#### 3. **코드 구현 핵심**
   ```c
   // FATFS로 SD 카드 파일 목록 읽기
   f_opendir(&dir, "/");
   while (f_readdir(&dir, &file_info) == FR_OK) {
       // 확장자 필터링
       if (is_hwp_or_doc(file_info.fname)) {
           // USB 메모리에 파일 복사
           copy_to_usb(file_info.fname);
       }
   }
   ```

### 📌 **왜 MSC 방식이 아닌가?**
- **MSC의 설계 목적**: 저장 장치를 **블록 디바이스**로 노출하는 것[2].  
- **파일 계층 추상화 부재**: 파일 시스템, 디렉토리, 확장자 개념이 프로토콜에 없음[3][4].  
- **호스트 의존성**: 모든 파일 메타데이터 처리는 PC에 의존적입니다.

### 💡 결론: **"된다" vs "안 된다"**
- **USB MSC 프로토콜만으로는 불가능**합니다[2][3].  
  → 호스트 OS가 파일 시스템을 제어하므로, 장치(STM32)의 필터링 개입 불가.  
- **STM32를 USB 호스트로 전환하면 가능**합니다.  
  → STM32가 SD 카드와 USB 메모리의 파일 시스템을 직접 관리할 때만 구현 가능.  

이 접근법은 **FATFS + USB Host 스택**을 활용해야 하며, STM32CubeMX에서 `USB_OTG_Host` 모드로 생성된 프로젝트 템플릿을 기반으로 개발할 수 있습니다.

[1] https://www.usenix.org/system/files/hotstorage20_paper_zhong_0.pdf
[2] https://www.reddit.com/r/embedded/comments/12oa1lb/at_what_abstraction_level_does_the_usb_msc_class/
[3] https://community.st.com/t5/stm32-mcus-products/usb-device-msc-with-external-flash/td-p/674099
[4] https://www.stm32duino.com/viewtopic.php?t=1077
[5] https://docs.vmware.com/en/VMware-Horizon/2309/horizon-remote-desktop-features/GUID-0BA5D4E1-B71C-4E24-BEFA-B6381D0E0BEF.html
[6] https://community.st.com/t5/stm32-mcus-embedded-software/problem-with-usb-mass-storage-device-class-on-stm32f401rbt6/td-p/296200
[7] https://docs.omnissa.com/bundle/Horizon-Remote-Desktop-FeaturesV2406/page/ConfiguringFilterPolicySettingsforUSBDevices.html
[8] https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/policies/reference/ica-policy-settings/usb-devices-policy-settings.html
[9] https://github.com/STMicroelectronics/stm32-usbx-examples
[10] https://www.youtube.com/watch?v=GjQqZd1keBo
[11] https://community.st.com/t5/stm32-mcus-embedded-software/usb-msc-device-low-transfer-rate-stm32f7/td-p/468009
[12] https://community.st.com/t5/stm32-mcus-products/is-it-possible-to-password-protect-an-usb-msc-drive-stm32l4/td-p/369473
[13] https://www.usb.org/sites/default/files/Mass_Storage_Specification_Overview_v1.4_2-19-2010.pdf
[14] https://e2e.ti.com/support/microcontrollers/arm-based-microcontrollers-group/arm-based-microcontrollers/f/arm-based-microcontrollers-forum/1124470/tm4c123ge6pm-help-usb-msc-vs-bulk-transfer-device
[15] https://controllerstech.com/stm32-usb-msc/
[16] https://embetronicx.com/tutorials/microcontrollers/stm32/stm32-usb-device-msc-using-flash-memory/
[17] https://controllerstech.com/stm32-usb-host-msc/
[18] https://www.youtube.com/watch?v=aEwwQMdKd-c
[19] https://github.com/LonelyWolf/stm32/blob/master/cube-usb-msc/msc/usbd_msc.c
