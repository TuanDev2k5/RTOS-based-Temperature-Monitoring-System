# RTOS-based Temperature Monitoring System

Dự án xây dựng hệ thống giám sát nhiệt độ đa nhiệm sử dụng vi điều khiển STM32F405RGTx và hệ điều hành thời gian thực FreeRTOS. Hệ thống tích hợp giao diện người dùng trên màn hình cảm ứng LCD, chuẩn giao tiếp mạng CAN và cơ chế quản lý năng lượng (Sleep/Wakeup).

Dự án được thực hiện trong khuôn khổ môn học Thực hành hệ thống nhúng – Trường Đại học Khoa học Tự nhiên (ĐHQG TP.HCM).

## Tổng quan tính năng

- **Giám sát thời gian thực:** Thu thập dữ liệu từ cảm biến nhiệt độ nội bộ (Internal ADC) của vi điều khiển.
- **Giao diện HMI (Human-Machine Interface):** Hiển thị và tương tác trực tiếp trên màn hình 2.8" Resistive Touch LCD (IC ST7789) thông qua chuẩn SPI.
- **Giao tiếp CAN Bus:** Xử lý truyền/nhận các khung dữ liệu nhiệt độ (Standard ID: `0x123`) giữa hai node mạng (CAN1 và CAN2).
- **Quản lý đa nhiệm:** Ứng dụng FreeRTOS để vận hành độc lập các tác vụ: thu thập cảm biến, cập nhật UI và xử lý sự kiện ngoại vi.
- **Power Management:** Tối ưu năng lượng bằng chế độ Sleep (WFI), hỗ trợ đánh thức bằng ngắt ngoài (EXTI).

## Yêu cầu phần cứng

- **Vi điều khiển (MCU):** STM32F405RGTx (Core Cortex-M4)
- **Màn hình:** 2.8 inch Resistive Touch LCD (ST7789)
- **Transceiver:** 2 Module giao tiếp CAN Bus (TJA1050 hoặc tương đương)

### Sơ đồ chân (Pin Mapping)

| Khối chức năng | Chân MCU | Mô tả chi tiết |
| :--- | :--- | :--- |
| **SPI1 (LCD & Touch)** | `PA5`, `PA6`, `PA7` | SCK, MISO, MOSI |
| **Điều khiển LCD** | `PB6`, `PB7`, `PB8` | Đèn nền (BL), Chọn chip (CS), Lệnh/Dữ liệu (DC) |
| **Cảm ứng (Touch)** | `PA9`, `PB4` | Chọn chip cảm ứng (TP_CS), Ngắt cảm ứng (TP_IRQ) |
| **CAN Bus 1** | `PA11`, `PA12` | CAN1_RX, CAN1_TX |
| **CAN Bus 2** | `PB12`, `PB13` | CAN2_RX, CAN2_TX |
| **Wakeup** | `PA0` | Nút nhấn phần cứng (Ngắt ngoài EXTI) |

## Kiến trúc phần mềm

Hệ thống được thiết kế dựa trên FreeRTOS với 3 luồng (Thread) chạy song song, trao đổi dữ liệu qua Message Queue:

1. `Temp_Task`: Lấy mẫu ADC kênh nhiệt độ nội mỗi 1000ms, tính toán nội suy ra nhiệt độ (°C) và đẩy dữ liệu vào `TempQueue`.
2. `UI_Task`: Nhận dữ liệu từ `TempQueue` và render giá trị lên màn hình LCD.
3. `Touch_Task`: Polling ngắt và đọc tọa độ chạm.
   - **Sự kiện PLAY:** Đóng gói payload nhiệt độ, phát qua CAN1 và kiểm tra buffer nhận của CAN2.
   - **Sự kiện PAUSE:** Tạm dừng SysTick và HAL Tick, gọi lệnh `WFI` (Wait For Interrupt) để đưa MCU vào trạng thái Sleep.

## Hướng dẫn Build & Flash

Dự án được cấu hình ngoại vi thông qua STM32CubeMX và có thể biên dịch bằng GNU Make hoặc STM32CubeIDE.

### 1. Biên dịch (Build)
Yêu cầu hệ thống đã cài đặt `arm-none-eabi-gcc` và `make`. Chạy các lệnh sau trong terminal:
```bash
cd Debug
make
