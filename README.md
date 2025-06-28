# 문서파일만 저장하는 USB MicroSD 
## Hardware: STM32H750VBT6 480MHz MPU Demo Board
- [유래텍 URAETECH](https://uraetech.co.kr/)
- [알리맨 유튜브](https://youtu.be/E8tZzawSQsM?si=naVZe0wMeBq0vCPa)

### STM32CubeIDE
- Menu > Manage Embedded Software Package
- STM32H 최신 버전 v170.zip 선택

## 두 가지 방식 
### MSC(Mass Storage Class): 파일 단위가 아님  https://youtu.be/_bfROydrssA?si=tHMm2wsZE3PyRl6v
### MTP(Media Transfer Protocol) 파일 단위 가능 함.

USB microSD 구현에서의 \*\*MSC(Mass Storage Class)\*\*와 \*\*MTP(Media Transfer Protocol)\*\*의 차이는 다음과 같습니다:

### 참고 동영상
https://youtu.be/I9KDN1o6924?si=F6ZpNetbuSHAzHi_

---

### 🔹 1. **접근 방식 차이**

| 구분        | **MSC (Mass Storage Class)** | **MTP (Media Transfer Protocol)** |
| --------- | ---------------------------- | --------------------------------- |
| **접근 방식** | 디스크 전체를 "블록 디바이스"로 마운트       | 파일 단위로 전송 (파일 시스템은 디바이스가 관리)      |
| **역할**    | 호스트(PC)가 파일 시스템에 직접 접근       | 디바이스가 파일 시스템을 관리하고 요청에 따라 전송      |

---

### 🔹 2. **파일 시스템 제어**

* **MSC**:

  * Windows가 microSD의 **파일 시스템을 직접 조작**
  * STM32 등 MCU는 동시에 파일 접근 **불가능** (파일 시스템 손상 위험 있음)

* **MTP**:

  * STM32가 파일 시스템을 **직접 관리** (FatFs 등 사용)
  * Windows는 STM32가 **허용한 파일만 접근 가능** → **파일 필터링 가능**

---

### 🔹 3. **파일 필터링 가능 여부**

| 기능                           | MSC   | MTP  |
| ---------------------------- | ----- | ---- |
| 특정 확장자만 저장 허용 (.hwp, .doc 등) | ❌ 어려움 | ✅ 가능 |
| 보안성 / 제어성                    | 낮음    | 높음   |

---

### 🔹 4. **운영체제 호환성 및 인식**

* **MSC**: 대부분의 OS에서 **USB 드라이브로 자동 인식**
* **MTP**: Windows는 기본 지원, macOS는 **추가 드라이버 필요**

---

### 🔹 5. **STM32에서 구현 난이도**

| 항목     | MSC                         | MTP                              |
| ------ | --------------------------- | -------------------------------- |
| 구현 난이도 | 낮음 (STM32CubeMX에서 간단 설정 가능) | 높음 (USBX or Custom MTP Stack 필요) |
| 코드 복잡도 | 낮음                          | 높음                               |

---

### 🔹 정리 요약

| 구분       | **MSC**        | **MTP**               |
| -------- | -------------- | --------------------- |
| 용도       | 단순 저장장치        | 미디어 파일 전송/제어          |
| 필터링 제어   | ❌ 없음           | ✅ 구현 가능               |
| PC 인식    | USB 드라이브       | MTP 장치                |
| STM32 접근 | 안 됨 (동시 접근 불가) | 가능 (STM32가 파일 시스템 소유) |
| 구현 난이도   | 쉬움             | 어려움                   |

---

### ✅ 사용 시 고려사항

* **보안** 또는 **파일 제한 기능이 필요**하면 → **MTP 추천**
* **단순 저장 기능만 필요**하면 → **MSC 추천**

필요하시면 STM32에서 MTP 구현 예제나 라이브러리 추천도 도와드릴 수 있습니다.


