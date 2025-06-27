좋습니다. STM32H750VBT6를 기반으로 한 프로젝트를 처음 접하는 사용자도 따라할 수 있도록, USBX 및 MTP 설정 방법, 확장자 필터링 로직, STM32CubeIDE 설정을 포함한 상세한 매뉴얼을 준비하겠습니다.

매뉴얼에는 단계별 설명, 예제 코드, 설정 스크린샷 등도 포함되며 `.hwp`, `.doc` 확장자만 저장되도록 하는 구체적인 구현 방법도 포함될 예정입니다.

준비가 완료되면 알려드리겠습니다.


# STM32H750VBT6 기반 USB MTP microSD (.hwp/.doc 전용) 저장 프로젝트 가이드

## 1. **하드웨어 설정**

* **STM32H750VBT6와 microSD 카드 연결**: 고속 전송을 위해 `SDMMC` 인터페이스를 사용합니다. SPI도 사용할 수 있지만 SDMMC(4-bit 모드)가 **성능 면에서 유리**하며 공식 권장 방식입니다. SDMMC 사용 시 아래 핀들을 연결해야 합니다:

  * SDMMC 핀 구성: `CK`(클럭), `CMD`(커맨드), 데이터 라인 `D0–D3`를 마이크로컨트롤러와 microSD 소켓에 연결해야 합니다. (STM32H7의 SDMMC는 4비트 모드 지원)
  * 추가로, 데이터 라인 및 CMD 라인에는 **풀업 저항**(내부 또는 10kΩ 정도) 적용이 필요합니다. 대부분 SD 소켓 모듈에 기본 풀업이 있지만, 없다면 GPIO 설정에서 Pull-Up 활성화 하십시오.
* **USB OTG 포트 설정**: 보드의 USB OTG 포트를 **디바이스 모드**로 사용합니다. STM32H750에는 `USB_OTG_FS`(내장 Full Speed PHY)와 `USB_OTG_HS`(고속, 외부 ULPI PHY 필요)가 있습니다. 적절한 포트를 디바이스로 활성화하세요 (예: FS 포트 사용 시 Mode를 *Device Only*로 설정). 고속 모드를 쓰려면 추가 PHY 하드웨어가 필요하므로, 처음에는 FS 모드를 권장합니다.

  * 회로상 **VBUS** 핀이 5V 입력을 인지해야 USB 디바이스가 인식되므로, 버스 파워를 사용하는 경우 VBUS 핀에 5V 연결을 잊지 마세요.
  * D+와 D- 데이터 라인에는 22Ω 시리즈 저항을 넣어 **임피던스 매칭**을 하는 것이 일반적입니다. 이는 신호 무결성을 높여 노이즈로 인한 연결 불안을 줄여줍니다.

## 2. **소프트웨어 스택 통합**

STM32CubeIDE의 CubeMX를 통해 **FatFs 파일시스템**, **USBX MTP 클래스**, **SDMMC 드라이버**를 통합합니다. 이 단계에서는 **Azure RTOS USBX의 MTP 디바이스 클래스**를 사용하여 표준 MTP 프로토콜 준수 디바이스를 구현합니다. 또한 FatFs를 통해 microSD에 **FAT32 파일 시스템**을 구성하여 PC와 파일을 주고받을 수 있게 합니다.

#### a) **FatFs + microSD 초기화**

먼저 microSD 카드를 FatFs로 제어하기 위한 초기화를 진행합니다. CubeMX에서 **FatFs 미들웨어**를 추가하고, Disk 드라이버는 `User Defined`로 설정합니다 (또는 SDMMC 전용 드라이버를 사용해도 무방함). 이렇게 하면 `USER_diskio.c` 등의 파일이 생성되며, SD 카드 제어를 위한 훅을 구현할 수 있습니다. 기본 절차는 다음과 같습니다:

```c
/* SDMMC 및 FatFs 초기화 예시 */
FATFS fs;    // FatFs 파일시스템 객체
FIL fil;     // 파일 객체 (파일 핸들)

// microSD 카드 마운트 시도
FRESULT res = f_mount(&fs, "", 0);
if (res != FR_OK) {
    // 필요시 FAT32로 파일시스템 생성 (포맷)
    f_mkfs("", FM_FAT32, 0, work_buf, sizeof(work_buf));
    f_mount(&fs, "", 0);  // 다시 마운트
}
```

* **FAT32 포맷**: Windows와 호환성을 위해 SD 카드를 FAT32로 포맷합니다. 새 카드이거나 exFAT 등으로 포맷된 경우 `f_mount`에 실패할 수 있으므로, 이때 `f_mkfs()`를 호출하여 FAT32로 재포맷합니다. (주의: 32GB를 초과하는 SDXC 카드는 기본 exFAT이므로 반드시 FAT32로 다시 포맷해야 PC에서 MTP 장치로 인식 및 읽기/쓰기가 원활합니다.)
* **SDMMC 속도 설정**: STM32H7의 SDMMC 클럭은 초기화 시 **400kHz 이하**로 설정되고, 데이터 전송 단계에서 최대 48MHz 정도로 사용합니다. 너무 높은 클럭은 안정성을 떨어뜨릴 수 있으므로 CubeMX에서 SDMMC 클럭 분주값을 충분히 크게 주어 (예: 분주율 4 이상의 설정) 초당 48MHz 이하로 동작하도록 합니다. 필요하면 SDMMC 초기 분주값(`Init Clock Divider`)을 높게 주어 인식 실패를 해결할 수 있습니다.
* **Disk I/O 드라이버 구현**: CubeMX의 User Defined 드라이버를 선택한 경우, `disk_read`, `disk_write`, `disk_ioctl` 등의 함수를 직접 구현해야 합니다. STM32H7 HAL의 SDMMC 드라이버(HAL\_SD\_...)를 호출하여 해당 함수를 작성하거나, STM32Cube FW에서 제공하는 예제 코드를 참고하여 작성합니다. (DeepBlue 임베디드 등 참고자료에 SD 카드 드라이버 예시가 있습니다.)
* **멀티스레드 고려**: USBX는 RTOS 환경에서 동작하므로, FatFs를 여러 스레드에서 접근할 때 re-entrancy 옵션을 활성화해야 합니다. CubeMX의 FatFs 설정에서 `_FS_REENTRANT`를 1로 설정하고, 적절한 동기화 객체를 설정하면(FatFs의 경우 semaphore를 자동 지원) 동시에 접근 시 문제를 방지할 수 있습니다.


*그림: FatFs를 통한 파일 접근 아키텍처. 애플리케이션은 FatFs API를 호출하고, FatFs는 디스크 I/O 드라이버를 통해 SD 카드(HAL SDMMC)로 접근합니다.*

#### b) **USBX + MTP 설정**

이제 **USBX 미들웨어**에서 MTP 클래스를 설정합니다. CubeMX의 *Middleware* 탭에서 **Azure RTOS ThreadX**와 **USBX Device**를 활성화해야 합니다. USBX Device 설정에서 **Class** 항목에 *PIMA MTP* (Media Transfer Protocol)를 추가합니다. CubeMX가 지원하지 않는 경우, ST가 제공한 X-CUBE-AZRTOS 패키지와 예제를 참고하여 수동 설정할 수 있습니다. CubeMX를 통해 설정할 경우 아래와 같은 초기 코드가 생성됩니다:

```c
// USBX Device Initialization (자동 생성 코드 일부 발췌)
VOID MX_USBX_Device_Init(VOID)
{
    // ... ThreadX, USB 장치 초기화 ...
    UX_DEVICE_CLASS_MTP_PARAMETER mtp_parameter;
    // MTP 클래스 파라미터 구조체 설정 (메모리 버퍼 등)
    ux_device_stack_class_register(_ux_system_device_class_mtp_name, 
                                   ux_device_class_mtp_entry, 1, 0x00, 
                                   &mtp_parameter);
}
```

* **ThreadX 활성화**: USBX는 Azure RTOS 환경에서 동작하므로 ThreadX 커널이 필요합니다. CubeMX에서 ThreadX를 활성화하면 자동으로 기본 스레드와 메모리 할당 코드가 추가됩니다. USBX Device도 ThreadX 상에서 초기화되며, MTP 클래스도 내부적으로 전용 스레드를 사용합니다.
* **MTP 클래스 등록**: `ux_device_stack_class_register()` 호출을 통해 MTP 클래스가 USB 디바이스에 등록됩니다. 이때 사용할 스토리지 등을 연결하기 위해 `UX_DEVICE_CLASS_MTP_PARAMETER` 구조체를 설정해야 하는데, CubeMX 생성 코드에서는 기본 값으로 설정되거나 별도 함수 호출로 구현자에게 위임될 수 있습니다. (예: `ux_device_class_mtp_activate` 콜백 등)
* **스토리지 연동**: MTP 클래스는 파일 단위 프로토콜이므로, PC로부터 받은 파일 데이터를 저장할 경로를 우리가 직접 처리해야 합니다. USBX MTP 장치에서 파일 전송 요청이 오면, 특정 콜백 함수를 통해 파일 생성/저장 동작을 구현하게 됩니다. 이 부분을 다음 단계에서 커스터마이징하여 microSD (FatFs)에 파일을 쓰도록 합니다.

> **참고:** USBX의 MTP 클래스는 **PTP (Picture Transfer Protocol)** 기반 Media Transfer Protocol 구현체입니다. PC(호스트)는 장치를 드라이브로 인식하지 않고 **휴대용 장치**로 인식하며, 파일 시스템 전체가 아닌 **파일 단위**로 통신합니다. 이 덕분에 장치가 파일 시스템의 무결성을 스스로 관리할 수 있고, 파일 전송 중 오류가 나도 장치 측 파일시스템이 손상되지 않는 장점이 있습니다. (MSC 방식은 호스트가 파일시스템을 직접 제어하므로 이러한 제어가 불가능)

## 3. **파일 필터링 (.hwp, .doc 확장자 제한)**

이 프로젝트의 핵심은 **특정 확장자 파일만 허용**하는 것입니다. MTP 프로토콜 자체에는 호스트 측에서 파일 형식을 제한하는 기능이 없으므로, **디바이스 펌웨어에서 파일 생성 요청을 가로채어 검사**해야 합니다. 이를 위해 **USBX MTP 클래스의 콜백**을 수정합니다.

예를 들어, USBX MTP 클래스에는 호스트가 파일을 보낼 때 호출되는 함수나 이벤트 콜백이 있습니다 (예: `ux_device_class_mtp_object_added` 또는 `UX_MTP_EVENT` 처리 루틴). 해당 부분에 아래와 같은 **확장자 검사 로직**을 추가합니다:

```c
UINT usr_mtp_object_received(UX_MTP_EVENT *event, UCHAR *object_buffer, ULONG obj_size) {
    CHAR *filename = event->ux_mtp_object_filename;
    CHAR *ext = strrchr(filename, '.');  // 파일명에서 확장자 추출
    if (ext && (strcasecmp(ext, ".hwp") == 0 || strcasecmp(ext, ".doc") == 0)) {
        // 허용된 확장자일 경우 -> 파일을 microSD에 저장
        FRESULT res = f_open(&fil, filename, FA_CREATE_ALWAYS | FA_WRITE);
        if (res == FR_OK) {
            UINT bytes_written;
            f_write(&fil, object_buffer, obj_size, &bytes_written);
            f_close(&fil);
            return UX_SUCCESS;
        } else {
            return UX_ERROR;  // 파일 열기 실패 시 에러 리턴
        }
    } else {
        // 허용되지 않은 확장자 -> 거부
        return UX_ERROR;  // 에러 코드를 반환하여 호스트에게 전송 거부
    }
}
```

위 코드에서는 전달된 `filename`의 확장자를 확인하여 `.hwp` 또는 `.doc`인 경우에만 FatFs를 통해 microSD에 파일을 생성/저장하고, 다른 경우 `UX_ERROR`를 반환합니다. `UX_ERROR`를 반환하면 USBX 스택은 해당 전송을 중단하고 호스트(PC)에 오류를 전달합니다. **Windows**의 경우 사용자가 `.txt`와 같은 금지된 파일을 복사 시도하면 \*"장치에서 파일을 저장할 수 없습니다"\*와 같은 메시지가 나오며 복사가 취소됩니다. 이처럼 장치 펌웨어가 **전송을 거부**함으로써 원하는 파일 종류만 저장되도록 강제할 수 있습니다. (MTP는 파일 수준 프로토콜이라 가능하며, Mass Storage 클래스였다면 이러한 개별 파일 거부가 불가능합니다.)

> **구현 팁:** 위 콜백 함수는 USBX MTP 클래스의 **데이터 이벤트** 안에서 호출되도록 등록해야 합니다. 예를 들어 USBX MTP 클래스의 parameter 구조체 (`UX_DEVICE_CLASS_MTP_PARAMETER`)의 `ux_device_class_mtp_instance_activate` 또는 유사한 콜백을 지정해 해당 함수가 동작하도록 합니다. CubeMX 생성 코드에서는 약간 다를 수 있으므로, 제공된 MTP 예제나 USBX 사용자 가이드를 참고하여 콜백 등록 부분을 확인하십시오. 콜백 내부에서 FatFs 함수(`f_open`, `f_write` 등)를 호출하므로, **FatFs가 미리 마운트**되어 있어야 하며, 동시에 파일 쓰기 중인 경우 없도록 적절한 배려(뮤텍스 등)도 고려됩니다.

## 4. **STM32CubeIDE 설정 요약**

이 절에서는 CubeMX에서 수행해야 할 설정들을 순서대로 정리합니다. 앞선 내용을 토대로 CubeIDE의 .ioc 설정 화면에서 다음을 확인하세요:

1. **RCC 클럭 설정**: USB와 SDMMC가 모두 정상 동작하려면, PLL 설정에서 48MHz 출력이 활성화되어야 합니다. `PLL3` 혹은 `PLLQ` 등을 사용하여 48MHz를 생성하고 USB 클럭 소스로 설정합니다 (CubeMX Clock Tree 탭 참고). SDMMC 클럭도 APB 버스 클럭 등을 통해 설정되는데, 초기화 클럭은 자동 조정되므로 크게 신경쓰지 않아도 됩니다.
2. **핀이설 및 주변장치 활성화**:

   * **SDMMC1** (또는 SDMMC2): 해당 SDMMC 인터페이스를 활성화합니다. Mode는 SDCard 4-bit Wide Bus로 설정하고, **DMA**를 추가로 설정하면 대용량 파일 전송 시 효율이 높아집니다. CubeMX의 Pinout에서 SDMMC에 필요한 핀들 (CMD, CK, D0\~D3)을 지정하고, 나머지 설정(속도, 풀업 등)은 자동 또는 수동으로 조정합니다. (H7의 경우 GPIO 설정에서 Pull-up을 `Enabled`로 설정 권장) 클럭분주 설정은 Parameter Settings에서 확인 가능하며, 필요시 높여줍니다.
   * **USB\_OTG\_FS** (또는 HS): USB\_OTG\_FS를 "Device\_Only" 모드로 설정합니다. CubeMX Pinout에서 DM, DP 핀 (FS의 경우 PA11(DM), PA12(DP))이 활성화되었는지 확인하세요. (HS를 사용할 경우 외부 PHY 설정 필요).
3. **Middleware - FatFs**: FatFs를 **Enabled**로 설정합니다. `Use Directories`, `Use Long File Names` 등 필요한 옵션을 활성화합니다. 가장 중요하게, **User-defined** 드라이버를 선택하거나, SDMMC 드라이버를 선택해야 합니다. 만약 `SDMMC` 드라이버 옵션이 CubeMX에서 제공된다면 선택하여도 됩니다 (CubeMX 버전에 따라 STM32H7 SDMMC 드라이버를 지원). 여기서는 User-defined로 가정하며, CubeMX가 `USER_diskio.c` 파일을 생성해 줄 것입니다.
4. **Middleware - Azure RTOS**: **ThreadX**와 **USBX Device**를 차례로 체크하여 활성화합니다. ThreadX 활성화 시 기본 설정 (최대 스레드 개수, 메모리 풀 크기 등)은 디폴트로 두고, USBX Device를 설정할 때 **Interface**로 USB\_OTG\_FS (또는 HS)를 선택합니다. **Class** 항목을 추가하고 드롭다운에서 **MTP** (또는 PIMA\_MTP) 클래스를 선택합니다.

   * 이때 FileX 미들웨어는 사용하지 않으므로 활성화 안 해도 됩니다. 우리가 FatFs를 사용할 것이기 때문에, USBX의 MTP 예제 코드 중 FileX 연동 부분이 있다면 제외하거나 무시합니다.
   * MTP 클래스 추가 후 Parameter 창에서 **Container ID**나 **메모리 버퍼** 설정 등이 있다면 요구대로 채워줍니다. (예: 오브젝트 최대 크기, 객체 리스트용 메모리 등. 기본값으로 일단 진행 가능)
5. **生成 및 컴파일**: CubeMX 설정을 마쳤으면 코드를 생성(Generate Code)하고, 프로젝트를 빌드합니다. STM32H750의 내부 SRAM이 1MB로 비교적 크지만, MTP 및 ThreadX, USBX를 사용하면 메모리 사용량이 많으므로 **Heap 및 Stack 크기**를 적절히 늘려줍니다. (예: Heap = 0x4000 정도부터 시작하여 부족 시 조절)

이상의 CubeMX 설정을 완료하면, 기본적인 프로젝트 뼈대가 만들어집니다. 생성된 코드에는 USBX MTP 클래스가 초기화되고, SD 카드도 HAL을 통해 초기화되는 코드가 포함될 것입니다. 이제 앞서 설계한 **파일 필터링 로직**을 해당 프로젝트에 추가해야 합니다. 구체적으로, `USBX_Device_MTP.c` 또는 유사한 파일/함수에 우리의 확장자 검사 코드를 삽입합니다. (CubeMX가 생성한 함수를 직접 수정하면 이후 재생성 시 덮어써질 수 있으므로, **USER CODE BEGIN/END** 구문 안에 코드를 추가하시기 바랍니다.)

## 5. **동작 테스트**

이제 PC와 연결하여 제대로 동작하는지 확인합니다:

1. **장치 인식 확인**: STM32 보드를 USB로 PC(Windows)에 연결합니다. 제대로 구현되었다면 Windows에서 "MTP 장치" 또는 지정한 제품 이름으로 인식됩니다. 파일 탐색기에서 장치가 **휴대용 장치**로 표시되어 폴더에 접근할 수 있어야 합니다. (드라이브 문자로 나타나지 않는 것이 정상입니다. MTP는 카메라나 휴대폰처럼 동작합니다.)
2. **허용 파일 전송 (.hwp/.doc)**: PC에서 `.hwp` 또는 `.doc` 파일을 장치로 복사해봅니다. 진행 막대가 표시되고, 전송이 완료되면 별도 오류 없이 사라지면 성공입니다. 복사한 후 microSD 카드를 PC에 직접 넣어 확인하거나, 펌웨어에서 해당 파일이 존재하는지 로그를 통해 확인할 수 있습니다. 파일이 제대로 저장되었다면 필터링 및 저장 기능이 정상입니다.
3. **차단 파일 전송 (.txt 등)**: 이번엔 `.txt`나 `.pdf` 등 허용되지 않은 파일을 복사 시도해봅니다. Windows 상에서 복사가 시작되다가 **오류 메시지**와 함께 중단되어야 합니다. 오류 내용은 "장치에서 이 파일을 저장할 수 없습니다" 또는 "디바이스가 응답하지 않습니다" 등의 형태일 수 있습니다. 이 동작은 우리의 펌웨어가 `UX_ERROR`를 반환하여 전송을 거부한 결과이며, microSD에는 해당 파일이 저장되지 않아야 합니다.
4. **여러 파일 및 폴더**: 폴더를 만들어 `.hwp` 파일 여러 개를 복사하거나, 허용/비허용 파일이 섞인 상황을 테스트합니다. 허용된 파일만 폴더 내에 저장되고, 나머지는 오류로 무시되는지 확인합니다. 또한 이미 microSD에 존재하는 다른 파일들을 PC에서 열람하거나 복사 시도할 때 정상 동작하는지 (읽기 기능)도 확인하십시오.

테스트 중 **MTP 전송 속도**는 Mass Storage보다는 느릴 수 있습니다. 이는 프로토콜 특성과 FatFs 처리 속도에 따른 것이므로, 너무 큰 파일 전송 시 시간이 걸릴 수 있음을 감안하세요.

## 6. **문제 해결 팁**

초기 구현 후 발생할 수 있는 자주 있는 문제와 해결책을 정리합니다:

* **SD 카드 마운트 실패/파일 시스템 인식 불가**: `f_mount`가 FR\_NO\_FILESYSTEM을 리턴한다면 파일시스템 없음 또는 호환 안 됨을 의미합니다. 이 경우 `f_mkfs`로 FAT32 포맷을 시도해보세요. 그래도 안된다면 전원 부족이나 배선 문제일 수 있으니 SD 카드 소켓 배선 (특히 GND 공유, VDD) 및 클럭 설정을 점검합니다. 또한 SDMMC 클럭을 너무 높게 설정한 경우 초기화 단계에서 오류가 날 수 있으므로, 분주 값을 높여 클럭을 낮춰보십시오.
* **USB 장치 인식 안됨**: PC에서 아예 MTP 장치로 인식하지 못한다면, USB OTG 설정이나 클럭 설정 문제일 수 있습니다. RCC에서 48MHz 공급이 되는지 확인하고, USB\_OTG\_FS global interrupt가 활성화되어 있는지, 그리고 USB DP 라인에 1.5K 풀업이 연결되어 있는지 점검합니다 (STM32H7의 FS PHY는 내장 풀업을 사용). 또한 **VBUS sensing**이 활성화되어 있는데 버스 전원이 제대로 안 들어오는 경우도 있으니 CubeMX 설정에서 VBUS Sense를 Disabled로 해보거나 회로를 확인합니다.
* **MTP 전송 도중 끊김**: 파일 복사 중 장치가 연결 해제되거나 오류가 발생한다면, **메모리 부족**이나 **전송 버퍼 문제**일 수 있습니다. CubeMX에서 할당된 MTP 클래스 버퍼 크기를 늘려보고, ThreadX의 기본 메모리 풀도 충분한지 확인합니다. 필요시 `.heap` 영역 크기를 늘립니다. 또한 USB 고속(HS) 사용 시 PHY 신호 무결성 문제일 수 있으므로 배선 길이를 최소화하고, D+/D-에 직렬 22Ω을 다시 확인하세요 (임피던스 매칭으로 신호 반사를 방지하여 안정성을 높임).
* **한글 파일 이름 문제**: FatFs 기본 설정으로는 DBCS/UTF-8 처리가 비활성화되어 있어 한글 이름 파일이 깨질 수 있습니다. 한글 파일명을 사용해야 한다면, CubeMX FatFs 옵션에서 **TCHAR 코드**를 `UTF-8` 또는 `Unicode`로 설정하고, `_USE_LFN`(Long File Name)을 1로 설정하세요. 이렇게 하면 한글 파일명도 처리 가능하지만, 약간의 코드 크기 증가가 있습니다.
* **기타**: MTP 장치 특성상 PC에서 파일을 여는 동작이 순차적으로 이뤄집니다. 대용량 파일을 열거나 복사할 때 **시간이 걸릴 수 있음**을 사용자에게 안내해야 할 수도 있습니다. 또한 MTP는 동시에 여러 파일 전송을 완벽히 병렬 처리하지는 못하므로, 파일을 하나씩 연속 복사하면 더 안정적입니다.

이상의 단계를 따르면, 처음 도전하는 분도 **STM32H750VBT6 기반 USB MTP microSD 저장장치** 프로젝트를 완성할 수 있을 것입니다. 요약하면, **Azure RTOS USBX의 MTP 클래스**와 **ChaN의 FatFs**를 연동하고, 응용 단계에서 **.hwp/.doc 파일 이외에는 저장 거부**하는 로직을 구현했습니다. STM32CubeIDE의 GUI 설정과 최소한의 코드 수정으로 해당 기능을 달성할 수 있으니, 참고 자료와 함께 따라 해보시기 바랍니다.

\[1] [https://blog.naver.com/specialist0/221010885789](https://blog.naver.com/specialist0/221010885789)
\[2] [https://deepbluembedded.com/stm32-sdmmc-tutorial-examples-dma/](https://deepbluembedded.com/stm32-sdmmc-tutorial-examples-dma/)
\[3] [https://deepbluembedded.com/stm32-sdio-sd-card-example-fatfs-tutorial/](https://deepbluembedded.com/stm32-sdio-sd-card-example-fatfs-tutorial/)
\[4] [https://deepbluembedded.com/stm32-sd-card-spi-fatfs-tutorial-examples/](https://deepbluembedded.com/stm32-sd-card-spi-fatfs-tutorial-examples/)
\[5] [https://www.youtube.com/watch?v=I9KDN1o6924](https://www.youtube.com/watch?v=I9KDN1o6924)
\[6] [https://community.st.com/t5/stm32-mcus-embedded-software/fatfs-using-usb-mass-storage-key/td-p/329513](https://community.st.com/t5/stm32-mcus-embedded-software/fatfs-using-usb-mass-storage-key/td-p/329513)
\[7] [https://www.youtube.com/watch?v=aEwwQMdKd-c](https://www.youtube.com/watch?v=aEwwQMdKd-c)
\[8] [https://01001000.xyz/2020-08-09-Tutorial-STM32CubeIDE-SD-card/](https://01001000.xyz/2020-08-09-Tutorial-STM32CubeIDE-SD-card/)
\[9] [https://community.st.com/t5/stm32-mcus-embedded-software/stm32l4-spi-sd-card-with-fatfs-and-usb-msc/td-p/131693](https://community.st.com/t5/stm32-mcus-embedded-software/stm32l4-spi-sd-card-with-fatfs-and-usb-msc/td-p/131693)
\[10] [https://nexp.tistory.com/2967?category=810135](https://nexp.tistory.com/2967?category=810135)
