[h7b0vbt6](detail-004.jpg)

---

## ✅ 세부 내용

### ✅ Core

* 32-bit Arm® Cortex®-M7 core (double-precision FPU, L1 cache)
* 16KB data + 16KB instruction cache (128-bit embedded Flash access)
* Up to 280 MHz, MPU, 599 DMIPS / 2.14 DMIPS/MHz (Dhrystone 2.1)
* DSP instructions 지원

---

### ✅ Memories

* 128KB Flash memory + 1KB OTP memory
* 약 1.4MB RAM:

  * 192KB TCM RAM (64KB ITCM + 128KB DTCM)
  * 1.18MB user SRAM
  * 4KB SRAM (Backup domain)
* 2× Octo–SPI (on-the-fly decryption, I/O multiplexing, etc.)

  * up to 40MHz (SRD), 110MHz (DTR mode)
* Flexible external memory controller (up to 32-bit):

  * SRAM, PSRAM, NOR Flash (up to 125MHz)
  * SDRAM/LPSDR SDRAM
  * 8/16-bit NAND Flash
* CRC calculation unit

---

### ✅ Security

* ROP, PC-ROP, active tamper
* Secure firmware upgrade
* Secure access mode

---

### ✅ General-purpose input/outputs

* 최대 138개 I/O (인터럽트 지원)
* Fast I/Os (최대 133 MHz)
* 최대 164개 5V-tolerant I/Os

---

### ✅ Reset and power management

* 2 전원 도메인 (CD, SRD):

  * CD: Cortex core 및 주변 장치용
  * SRD: reset, clock, power management용
* 1.62V \~ 3.6V operation
* POR, PDR, PVD, BOR
* USB PHY용 3.3V 내부 전원
* SDMMC 전원, 고효율 SMPS step-down converter
* LDO, backup regulator (\~0.9V)
* Sleep, Stop, Standby 모드
* VBAT 배터리 모드 (charging 지원)
* 전원 상태 모니터링 핀

---

### ✅ Low-power consumption

* Stop 모드: 32μA (RAM retention)
* Standby 모드: 2.8μA (Backup SRAM OFF, RTC/LSE ON, PDR OFF)
* VBAT 모드: 0.8μA (RTC, LSE ON)

---

### ✅ Clock management

* Internal oscillator:

  * 64MHz HSI, 48MHz HSI48, 4MHz CSI, 32kHz LSI
* External oscillator: 4\~50MHz HSE, 32.768kHz LSE
* 3× PLLs (fractional mode 포함)

---

### ✅ Interconnect matrix

* 3 bus matrix: 1 AXI, 2 AHB
* Bridges: 5× AHB2APB, 3× AXI2AHB

---

### ✅ 5 DMA controllers

* 1× MDMA
* 2× Dual-port DMA (FIFO 포함)
* 1× Basic DMA (request router)
* 1× Basic DMA (DFSDM 전용)

---

### ✅ Up to 35 communication peripherals

* 4× I2C FM+ (SMBus/PMBus)
* 5× USART/5× UART (1× LPUART 포함, LIN, IrDA, ISO7816 등)
* 6× SPI (4 full-duplex I2S 포함)
* 2× SAI (serial audio interface)
* SPDIFRX
* SWPMI (single-wire master)
* MIDI slave
* 2× SD/SDIO/MMC (최대 133 MHz)
* 2× CAN FD (1× time-triggered 포함)
* 1× USB OTG (1HS/1FS)
* HDMI-CEC
* 1\~14bit 카메라 인터페이스 (최대 80 MHz)
* 8/16-bit 병렬 synchronous data I/F (PSSI)

---

### ✅ 11 analog peripherals

* 2× ADC (16-bit, 최대 3.6 MSPS, 24채널 지원)
* 2× DAC (12-bit)
* 2× operational amplifier
* 2× digital filter for sigma delta modulator (DFSDM)
* 2× 8ch/8filter, 1× 2ch/1filter

---

### ✅ Graphics

* LCD-TFT controller (XGA 해상도)
* Chrom-ART DMA2D accelerator
* Hardware JPEG codec
* Chrom-GRC™ (GFXMMU)

---

### ✅ Up to 19 timers and 2 watchdogs

* 2× 32-bit 타이머 (IC/OC/PWM, encoder input)
* 2× 16-bit motor control timers
* 10× 16-bit general-purpose timers
* 3× 16-bit low-power timers
* 2× watchdogs (independent, windowed)
* 1× SysTick timer
* RTC (하드웨어 calendar 포함)

---

### ✅ Cryptographic acceleration

* AES: ECB, CBC, CTR, GCM, CCM (128/192/256-bit)
* HASH: MD5, SHA-1, SHA-2, HMAC
* 2× OTFDEC AES-128 CTR (Octo–SPI 메모리용)
* 1× 32-bit, NIST SP 800-90B 참 난수 생성기

---

### ✅ Debug mode / 96-bit unique ID

* SWD & JTAG 인터페이스
* 4KB Embedded Trace Buffer

---

필요하신 부분만 요약해드리거나, 엑셀 표로 재정리도 가능합니다. 원하시나요?
