# STM32 SD Card Data Logger (ADC + RTC)

This project implements a **data logger system** using an STM32 microcontroller.  
It periodically reads an analog value (ADC) and logs it together with timestamp information (RTC) to an SD card using FATFS.

---

## 🚀 Features

- 📊 ADC data acquisition (12-bit resolution)
- ⏱ RTC-based timestamping (HH:MM:SS + Date)
- 💾 SD card logging via SPI (FATFS)
- 📝 Automatic file creation and append (`Datalogger.txt`)
- 🔁 Continuous logging every 1 second

---

## 🧩 Hardware Requirements

- STM32 MCU (CubeMX configured)
- SD Card module (SPI interface)
- Analog input signal (connected to ADC Channel 6)
- RTC clock source (LSE recommended)

---

## 📂 Project Structure


.
├── Core/
│ ├── Src/main.c
│ ├── Inc/main.h
├── FATFS/
├── Drivers/
├── README.md


---

## ⚙️ Peripherals Used

| Peripheral | Purpose |
|----------|--------|
| ADC1 | Analog signal reading |
| RTC | Time & date tracking |
| SPI1 | SD card communication |
| FATFS | File system handling |

---

## 🔧 Configuration Details

### ADC
- Resolution: 12-bit
- Channel: `ADC_CHANNEL_6`
- Mode: Continuous conversion

### RTC
- Format: 24-hour
- Clock Source: LSE (Low Speed External)
- Prescalers: Asynch = 127, Synch = 255

### SPI
- Mode: Master
- Prescaler: 32
- Data Size: 8-bit

---

## 📝 Logging Format

### File: `Datalogger.txt`

#### Header:

Date=DD.MM.YY
Analog Value--------hour


#### Data:

0123 11:50:01
0456 11:50:02


---

## 🔄 How It Works

1. System initializes peripherals (ADC, RTC, SPI, FATFS)
2. Current date is read from RTC and written to file
3. Every 1 second:
   - ADC value is read
   - Current time is read
   - Data is formatted and appended to SD card

---

## ▶️ Main Loop Logic

```c
HAL_Delay(1000);

HAL_RTC_GetTime(&hrtc, &sTime, RTC_FORMAT_BIN);
adc_data = ADC1->DR;

sprintf(line2, "%0.4d  %0.2d:%0.2d:%0.2d\n", adc_data, hour, minute, second);

f_open(&fil, "Datalogger.txt", FA_OPEN_ALWAYS | FA_WRITE | FA_READ);
f_lseek(&fil, fil.fptr);
f_puts(line2, &fil);
f_close(&fil);

⚠️ Important Notes
ADC is read directly via register (ADC1->DR)
⚠️ Make sure ADC conversion is started (HAL_ADC_Start()) if needed
File is opened and closed every cycle
This is safe but not optimal for performance
RTC is initialized with fixed values
Consider backup register check to avoid reset on reboot
FATFS mount is called repeatedly
Can be optimized (mount once)
🛠 Improvements (Recommended)
 Use DMA for ADC
 Mount filesystem once (not every loop)
 Keep file open for faster writes
 Add error handling for SD card failures
 Add UART debug output
 Add file size limit / log rotation
 Use interrupt or timer instead of HAL_Delay
🐞 Known Limitations
No SD card removal detection
No power-fail protection
No buffering (risk of data loss on sudden reset)

📜 License

This project is based on STM32 HAL libraries (BSD 3-Clause License).

User code additions are free to use for educational and development purposes.