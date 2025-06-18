# SPWM FPGA Design

本專案在 FPGA 上實作正弦波脈寬調變（SPWM, Sinusoidal Pulse Width Modulation），並透過 **PLL** 和 **除頻器** 精準產生近似 60Hz 的 SPWM 波形輸出。


## 需求
* 當正弦波到達最高點（sin =+1）時 → 輸出的 PWM 是「完全高電位」 ➜ duty = 100%
* 當正弦波到達最低點（sin = -1）時 → 輸出的 PWM 是「完全低電位」 ➜ duty = 0%
* 其他時候 duty 介於 0～100% 之間 ➜ duty ∝ sin 值（比例關係）  
![image](https://github.com/user-attachments/assets/b1d5d811-94a3-4d75-aa0d-e2396e067276)
## Breakdown
![image](https://github.com/user-attachments/assets/2c41e235-3c52-499c-8ba4-d0345e38b243)


## 規格
* sine 頻率 : 60Hz

#### 系統環境

| 項目             | 說明                                      |
|------------------|-------------------------------------------|
| 開發板           | EGO-XZ7                                   |
| 系統時脈         | 100 MHz（FPGA 輸入主時脈）                |
| 正弦波產生方式   | 使用 256 筆 sine LUT 查表產生             |
| 正弦波資料格式   | `std_logic_vector(7 downto 0)`            |


#### 頻率與公式計算

| 項目                                 | 單位：Hz           | 計算公式                                   | 備註說明                                     |
|--------------------------------------|--------------------|---------------------------------------------|----------------------------------------------|
| 除頻後時脈（1個 PWM 週期）           | **24.4 KHz**       | `1 / (10ns × 4096)`                          | 每個 PWM 週期時間（divide(12) = 2¹² = 4096） |
| SPWM 最慢頻率（256 PWM × 1024）     | **0.0466 Hz**      | `1 / (10ns × 4096 × 256 × 1024)`            | `SIN_UPDATE_PERIOD = 1024`（最慢情況）       |
| SPWM 最快頻率（256 PWM × 1）        | **約 95 Hz**       | `1 / (10ns × 4096 × 256)`                   | `SIN_UPDATE_PERIOD = 1`（最快情況）          |

###### 說明：

- 一個完整正弦週期 = 256 筆 PWM 輸出
- 每筆 PWM 時間 = 4096 個 clock（@100MHz 時脈）＝約 40.96 μs
- 總正弦週期時間 = 256 × 40.96 μs × `SIN_UPDATE_PERIOD`
- 可以透過調整 `SIN_UPDATE_PERIOD` 來改變 SPWM 頻率

## API
| 名稱          | 類型     | 說明                              |
| ----------- | ------ | ------------------------------- |
| `i_clk`     | input  | 系統時脈，來源為 100MHz（經 PLL 處理）       |
| `i_rst`     | input  | 非同步重置信號，**低電位有效**               |
| `o_pwm_out` | output | SPWM 輸出訊號，根據 sine LUT 結果調整 duty |

##  架構

| 模組                   | 說明                                          |
|------------------------|-----------------------------------------------|
| `SPWM_top.vhd`         | 頂層模組，串接 PLL、除頻器、SPWM_main         |
| `SPWM_clk_divider.vhd` | 除頻模組，將 1MHz 時脈除以 4096（divide(12)） |
| `SPWM_main.vhd`        | 核心 SPWM 控制模組，內含 LUT、FSM、PWM 邏輯   |
| `design_1_wrapper.vhd` | Block Design 自動產生，內含 PLL（1MHz）      |
| `clk_wiz_0`            | Vivado Clocking Wizard 產生的 PLL IP        |
| `SPWM_test.xdc`        | 約束檔，設定輸入時脈與 PWM 輸出腳位           |
| `uart_rx.vhd`           | UART 接收模組，負責從 PS 端或外部終端機接收字元|
| `axi_interface.vhd`     | AXI-Burst 介面模組，接收 PS 指令並存取控制暫存器 |


## 驗證
![image](https://github.com/user-attachments/assets/313c4359-60e6-4936-904a-542b4c408e7e)
![image](https://github.com/user-attachments/assets/313c4359-60e6-4936-904a-542b4c408e7e)


## PS 端功能 (Processing System)

### 功能概述
- 提供 SPWM 頻率調整與控制功能
- 控制 PWM 輸出頻率
- 接收 UART 指令，控制 PL 中 SPWM 模組，並回傳狀態

### 規格
- **UART**
  - 
## API
| 名稱          | 類型     | 說明                              |
| ----------- | ------ | ------------------------------- |
| `i_clk`     | input  | 系統時脈，來源為 100MHz（經 PLL 處理）       |
| `i_rst`     | input  | 非同步重置信號，**低電位有效**               |
| `o_pwm_out` | output | SPWM 輸出訊號，根據 sine LUT 結果調整 duty |

##  架構

| 模組                   | 說明                                          |
|------------------------|-----------------------------------------------|
| `SPWM_top.vhd`         | 頂層模組，串接 PLL、除頻器、SPWM_main         |
| `SPWM_clk_divider.vhd` | 除頻模組，將 1MHz 時脈除以 4096（divide(12)） |
| `SPWM_main.vhd`        | 核心 SPWM 控制模組，內含 LUT、FSM、PWM 邏輯   |
| `design_1_wrapper.vhd` | Block Design 自動產生，內含 PLL（1MHz）      |
| `clk_wiz_0`            | Vivado Clocking Wizard 產生的 PLL IP        |
| `SPWM_test.xdc`        | 約束檔，設定輸入時脈與 PWM 輸出腳位           |
| `uart_rx.vhd`           | UART 接收模組，負責從 PS 端或外部終端機接收字元|
| `axi_interface.vhd`     | AXI-Burst 介面模組，接收 PS 指令並存取控制暫存器 |


## PS 端功能 (Processing System)

### 功能概述
- 提供 SPWM 頻率調整與控制功能
- 控制 PWM 輸出頻率
- 接收 UART 指令，控制 PL 中 SPWM 模組，並回傳狀態

### 規格
- **UART**
  - 鮑率：9600 / 115200（可選）
  - 支援指令：
    - `SET_FREQ_60`：設定為 60Hz 模式
    - `START`：啟動 PWM 輸出
    - `STOP`：停止 PWM 輸出
- **AXI-Burst**
  - 控制暫存器：提供控制訊號給 PL 端
  - 資料暫存器：寫入 SPWM 對應參數給 PL

## PL 端功能 (Programmable Logic)

### 功能概述
- 儲存 `sin_index` 資料，產生對應 PWM 輸出
- 根據 LUT 查表計算 PWM duty cycle
- 每次完整循環包含 256 筆資料（即 256 組 PWM 輸出）

### 規格
- **Clock：100MHz**
- **LUT：256 筆 duty 值資料**
- **PWM 控制：使用 sin_index (0~255) 進行輸出**
- **輸出：PWM 控制腳位**
- **介面：AXI-Burst 與 PS 端溝通**

