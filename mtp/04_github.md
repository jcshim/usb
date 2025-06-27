STM32H750VBT6 기반의 USB MTP + microSD(.hwp/.doc 전용) 프로젝트와 가장 밀접하게 관련된 GitHub 리포지토리는 다음과 같습니다:

---

## 🔧 1. **STMicroelectronics/stm32-usbx-examples**

* USBX 기반의 다양한 USB 디바이스 및 호스트 예제가 포함된 공식 예제 모음입니다.
* 특히 **PIMA/MTP** 디바이스 예제도 함께 제공되어, 해당 플랫폼에서 MTP 적용 흐름을 직접 확인할 수 있습니다 ([github.com][1], [github.com][2]).

### 주목 예제:

* `Projects/.../Applications/USBX/Ux_Device_PIMA_MTP/` – STM32H7 보드 기준 **MTP 디바이스** 구현 예제 ([github.com][3]).

---

## 📦 2. **STMicroelectronics/x-cube-azrtos-h7**

* STM32H7 시리즈용 Azure RTOS(USBX 포함) 통합 확장 패키지입니다.
* `Projects/STM32H735G-DK/Applications/USBX/Ux_Device_PIMA_MTP` 예제가 포함되어 있어 MTP 구현과 microSD 저장 연동 시 참조하기 좋습니다 ([github.com][4]).

---

## ✅ 추천 리포지토리 요약

| 리포지토리                   | 설명                     | 특징                               |
| ----------------------- | ---------------------- | -------------------------------- |
| **stm32-usbx-examples** | Azure RTOS USBX 예제 전체  | MTP 클래스 기반 예제가 포함된 USBX 예제 모음    |
| **x-cube-azrtos-h7**    | STM32H7 전용 RTOS 확장 패키지 | MTP + microSD 또는 FatFs 통합 가능성 존재 |

---

## 🧭 활용 가이드

1. **STM32CubeIDE 프로젝트에 `stm32-usbx-examples` 클론**

   * `Ux_Device_PIMA_MTP` 예제를 복사해서 STM32H750VBT6로 포팅
   * USBX MTP 초기화, 콜백 구조, descriptor 설정 흐름 확인 가능

2. **microSD 저장 연동**

   * 프로젝트에 FatFs(microSD) 코드를 추가
   * MTP 데이터 콜백 안에서 파일 확장자를 체크하고 `.hwp`/`.doc`만 저장하도록 로직 통합

3. **x-cube-azrtos-h7 참고**

   * USBX + ThreadX + FatFs 통합 예제 흐름 이해에 효과적
   * 필요한 부분을 포팅하거나 샘플 로직 참고용으로 활용

---

### ✅ 다음 단계 제안

* `stm32-usbx-examples`의 MTP 예제(`Ux_Device_PIMA_MTP`) 소스 확인
* USBX 콜백 구조 및 MTP 오브젝트 수신 API 분석
* FatFs 기반 microSD 저장 로직과 결합

이 리포지토리들을 참고하면, **USBX 기반 MTP + .hwp/.doc 전용 필터링 + microSD 저장** 구성을 훨씬 수월하게 구현할 수 있습니다. 더 필요한 예제 파일이나 코드 로직이 있으면 언제든 요청해주세요!

[1]: https://github.com/STMicroelectronics/stm32_mw_usb_device/issues/13?utm_source=chatgpt.com "Example of USB MTP class usage · Issue #13 - GitHub"
[2]: https://github.com/STMicroelectronics/stm32-usbx-examples?utm_source=chatgpt.com "STMicroelectronics/stm32-usbx-examples: Provide a set of ... - GitHub"
[3]: https://github.com/STMicroelectronics/stm32-usbx-examples/blob/main/Projects/STM32H735G-DK/Applications/USBX/Ux_Device_PIMA_MTP/README.md?utm_source=chatgpt.com "stm32-usbx-examples/Projects/STM32H735G-DK/Applications ..."
[4]: https://github.com/STMicroelectronics/x-cube-azrtos-h7?utm_source=chatgpt.com "X-CUBE-AZRTOS-H7 (Azure RTOS Software Expansion for ... - GitHub"
