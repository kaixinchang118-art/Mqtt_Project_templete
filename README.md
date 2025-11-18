# STM32 OLED Display Project

## Project Overview
This is an STM32F103-based IoT project that integrates an OLED display, DHT11 temperature/humidity sensor, ESP8266 WiFi module, and connects to the OneNet cloud platform via MQTT protocol. The system displays real-time sensor data on the OLED screen and synchronizes data with the cloud.

## Hardware Requirements

### Microcontroller
- **STM32F103** series (STM32F103xB)
- Clock: 72MHz (HSE with PLL)

### Peripherals
- **OLED Display**: 0.69" 128×64 pixels, SSD1306 controller (I2C interface)
- **DHT11 Sensor**: Temperature and humidity sensor
- **ESP8266 Module**: WiFi module for cloud connectivity
- **LED**: Status indicator
- **Key/Button**: User input

## Pin Configuration

### OLED Display (I2C)
| OLED Pin | STM32 Pin | Description |
|----------|-----------|-------------|
| GND      | GND       | Ground      |
| VCC      | 3.3V/5V   | Power Supply|
| SCL      | PB1       | I2C Clock   |
| SDA      | PB0       | I2C Data    |

### Communication Interfaces
- **USART1**: Primary serial communication
- **USART3**: ESP8266 WiFi module communication
- **TIM4**: Timer for DHT11 and delays

## Project Structure

```
10_OLED/
├── Bsp/                    # Board Support Package
│   ├── oled.c/h           # OLED display driver
│   ├── oledfont.h         # Font library for OLED
│   ├── dht11.c/h          # DHT11 sensor driver
│   ├── led.c/h            # LED control
│   └── key.c/h            # Key/button control
├── Com/                    # Common utilities
│   └── delay.c/h          # Delay functions
├── Core/                   # STM32 HAL core files
│   ├── Inc/               # Header files
│   └── Src/               # Source files
│       └── main.c         # Main application code
├── Drivers/                # STM32 HAL drivers
│   ├── CMSIS/             # CMSIS core
│   └── STM32F1xx_HAL_Driver/
├── Net/                    # Network components
│   ├── CJSON/             # JSON parser
│   ├── device/            # Device drivers
│   │   └── esp8266.c/h    # ESP8266 WiFi module
│   ├── MQTT/              # MQTT protocol implementation
│   └── onenet/            # OneNet cloud platform integration
└── CMakeLists.txt         # CMake build configuration
```

## Features

### OLED Display Functions
- **Display Temperature**: Shows real-time temperature in Celsius (℃)
- **Display Humidity**: Shows relative humidity percentage (%)
- **LED Status**: Displays LED on/off status in Chinese characters (亮/灭)
- **System Messages**: Connection status and initialization messages

### Display API Functions
```c
void OLED_Init(void);                           // Initialize OLED
void OLED_Clear(void);                          // Clear screen
void OLED_Display_On(void);                     // Turn on display
void OLED_Display_Off(void);                    // Turn off display
void OLED_ShowString(uint8_t x, uint8_t y, char *p, uint8_t size);
void OLED_ShowChar(uint8_t x, uint8_t y, uint8_t chr, uint8_t size);
void OLED_ShowNum(uint8_t x, uint8_t y, uint32_t num, uint8_t len, uint8_t size);
void OLED_ShowCHinese(uint8_t x, uint8_t y, uint8_t no);
void OLED_DrawBMP(uint8_t x0, uint8_t y0, uint8_t x1, uint8_t y1, uint8_t BMP[]);
void OLED_Set_Pos(uint8_t x, uint8_t y);       // Set cursor position
```

### Cloud Connectivity
- **Protocol**: MQTT over TCP
- **Platform**: OneNet (China Mobile IoT Platform)
- **Server**: mqtts.heclouds.com:1883
- **Features**:
  - Sends temperature and humidity data every 5 seconds
  - Receives remote commands (e.g., LED control)
  - Automatic reconnection on connection loss

## Building the Project

### Using CMake
```bash
# Configure the project
cmake -B build -DCMAKE_BUILD_TYPE=Debug

# Build the project
cmake --build build

# Flash to device
cmake --build build --target flash
```

### Prerequisites
- CMake 3.15 or higher
- ARM GNU Toolchain (arm-none-eabi-gcc)
- OpenOCD or STM32CubeProgrammer for flashing

## Usage

### Initialization Sequence
1. System HAL initialization
2. Clock configuration (72MHz)
3. GPIO, USART, and Timer initialization
4. LED and Key initialization
5. OLED initialization and display
6. DHT11 sensor initialization
7. ESP8266 WiFi module initialization
8. MQTT connection to OneNet server
9. Device login and subscription

### Runtime Operation
- The system continuously reads temperature and humidity from DHT11
- Data is displayed on OLED in real-time
- Sensor readings are sent to OneNet cloud every 5 seconds
- The system listens for incoming commands from the cloud
- LED status (on/off) is reflected on the OLED display

### Display Layout
```
温度：XX ℃
(Temperature: XX °C)

湿度：XX %
(Humidity: XX %)

台灯：亮/灭
(Lamp: On/Off)
```

## Configuration

### OneNet Connection
Modify the connection string in `main.c`:
```c
#define ESP8266_ONENET_INFO "AT+CIPSTART=\"TCP\",\"mqtts.heclouds.com\",1883\r\n"
```

### OLED I2C Address
Default slave address: `0x78` (defined in `oled.c`)

### Display Size
- Width: 128 pixels
- Height: 64 pixels (8 pages × 8 rows)
- Font sizes: 8×16 or 6×8 pixels

## Code Example

### Basic OLED Usage
```c
// Initialize OLED
OLED_Init();
OLED_Clear();

// Display strings
OLED_ShowString(0, 0, "Hello STM32", 16);

// Display numbers
OLED_ShowNum(0, 2, 12345, 5, 16);

// Display Chinese characters
OLED_ShowCHinese(0, 4, 0);  // Display defined Chinese character
```

### Reading and Displaying Sensor Data
```c
uint8_t temp, humi;

// Read DHT11 sensor
DHT11_Read_Data(&temp, &humi);

// Display on OLED
char buf[4];
sprintf(buf, "%2d", temp);
OLED_ShowString(54, 0, buf, 16);  // Temperature value

sprintf(buf, "%2d", humi);
OLED_ShowString(54, 3, buf, 16);  // Humidity value
```

## Troubleshooting

### OLED Not Displaying
- Check I2C connections (SCL on PB1, SDA on PB0)
- Verify power supply (3.3V or 5V)
- Ensure proper initialization timing (800ms delay)
- Check I2C slave address (0x78)

### DHT11 Reading Failures
- Verify DHT11 is properly connected
- Check timing configuration
- Ensure TIM4 is initialized correctly

### ESP8266 Connection Issues
- Verify USART3 configuration and baud rate
- Check AT command responses
- Ensure WiFi credentials are correct
- Verify OneNet platform credentials

## References

- [STM32F103 Reference Manual](https://www.st.com/resource/en/reference_manual/cd00171190.pdf)
- [SSD1306 OLED Controller Datasheet](https://cdn-shop.adafruit.com/datasheets/SSD1306.pdf)
- [DHT11 Sensor Datasheet](https://www.mouser.com/datasheet/2/758/DHT11-Technical-Data-Sheet-Translated-Version-1143054.pdf)
- [OneNet Platform Documentation](https://open.iot.10086.cn/)

## License

This project uses code from various sources:
- OLED driver adapted from 中景园电子 (Zhongjingyuan Electronics)
- STM32 HAL library from STMicroelectronics
- MQTT and OneNet integration libraries

Please refer to individual source files for specific license information.

## Author

- Original OLED driver: Evk123 (中景园电子)
- Project integration: [Your Name]

## Version History

- **v2.0**: Current version with OneNet cloud integration
- Display features: Temperature, humidity, and LED status
- Communication: MQTT over ESP8266

---

**Note**: This project is designed for educational and learning purposes. Please ensure proper hardware connections before powering on the system.
