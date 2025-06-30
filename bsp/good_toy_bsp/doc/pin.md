# 玩具硬件完整引脚配置 (基于ESP32-S3)

> **🎵 音频架构特点**: 采用ES7210(ADC录音) + ES8311(DAC播放)双芯片分离式设计，提供专业级音频处理能力和更低的信号串扰。

## 音频系统 (双芯片架构)

### ES7210 ADC芯片 (录音专用):
#### I2C控制接口:
- SDA: GPIO17 (ES_IIC_SDA)
- SCL: GPIO18 (ES_IIC_SCL)
- I2C地址: 0x40 (ES7210_AD1_AD0_STRAP)

#### I2S录音数据输出:
- MCLK: GPIO16 (ES7210_MCLK/ADDA_MCLK/CODEC_IIS_MCLK) - 主时钟输入到ES7210
- SCLK: GPIO9 (ES7210_SCLK/ADDA_SCLK/CODEC_IIS_SCLK) - 串行时钟  
- LRCK: GPIO45 (ES7210_LRCK/ADDA_LRCK/CODEC_IIS_LRCK) - 左右声道时钟
- SDOUT: GPIO10 (ES7210_SDOUT) - 录音数据输出

#### 麦克风输入:
- ADC_MICN: 麦克风负极输入 (通过C50电容耦合)
- ADC_MICP: 麦克风正极输入 (通过C62电容耦合)
- MIC_BIAS: 麦克风偏置电压输出

### ES8311 DAC芯片 (播放专用):
#### I2C控制接口:
- SDA: GPIO17 (ES_IIC_SDA) - 共享I2C总线
- SCL: GPIO18 (ES_IIC_SCL) - 共享I2C总线
- I2C地址: 0x18 (ES8311_IIC_ADDR)

#### I2S播放数据输入:
- MCLK: GPIO16 (CODEC_IIS_MCLK) - 主时钟
- SCLK: GPIO9 (CODEC_IIS_SCLK) - 串行时钟
- LRCK: GPIO45 (CODEC_IIS_LRCK) - 左右声道时钟  
- DSDIN: GPIO8 (CODEC_IIS_DSDIN) - 播放数据输入

#### 模拟音频输出:
- DAC_OUTP: ES8311模拟音频正极输出 → NS4150B功放输入
- DAC_OUTN: ES8311模拟音频负极输出 → NS4150B功放输入

## 用户接口
### 按键控制:
- KEY_VOL-: GPIO5 (音量减按键)  
- KEY_VOL+: GPIO6 (音量加按键)

### 指示灯:
- WS2812_DAT: GPIO38 (可编程RGB LED控制)

## 功率管理
### 音频功放:
- PA_CTRL: GPIO48 (功率放大器控制)

### 充电管理:
- CHG_STA: GPIO22 (充电状态检测)

## 通信接口  
### WiFi控制:
- H_INT: GPIO1 (中断信号)

### USB接口:
- USB1_DP: GPIO19 (USB D+)
- USB1_DM: GPIO20 (USB D-)

## 外设扩展
### 电源管理芯片(CW2215):
- CW2215_SDA1: GPIO11 (数据线)
- CW2215_SCL1: GPIO12 (时钟线)  
- CW2215_INT1: GPIO13 (中断)

### 系统控制:
- ESP32_EN: 系统使能  
- BOOT0A: GPIO0 (下载模式)
- VCC_ESP2_ctrl: 电源控制

### 扩展IO:
- ESP_TO_ESP_IO1: GPIO42
- ESP_TO_ESP_IO2: GPIO41
- ESP_TO_ESP_IO3: GPIO40
- ESP_TO_ESP_IO4: GPIO39
- ESP_TO_ESP_IO5: GPIO7
- ESP_TO_ESP_IO6: GPIO15

## 音频参数配置 (双芯片分工)
- 采样率: 16kHz
- 位深度: 16-bit  
- 声道: 单声道 (单麦克风)
- **录音芯片**: ES7210 ADC (专业4路ADC，使用其中1路)
- **播放芯片**: ES8311 DAC (音频编解码器的DAC功能)
- 麦克风增益: 30dB (可调，ES7210控制)
- 功放: NS4150B (接收ES8311的DAC_OUTP/DAC_OUTN信号)
- **设计优势**: 分离式设计提供更好的音频质量和更低的串扰

## 🚨 重要说明和测试状态

### ✅ 已验证功能
- **音量控制按键** (2024-06-29):
  - GPIO5 (音量减): 测试正常 ✅
  - GPIO6 (音量加): 测试正常 ✅  
  - 音量范围: 0%-100% (5%步进)
  - 消抖功能: 50ms 正常工作
  - 按键响应时间: ~100-250ms

- **WS2812B LED诊断系统** (2024-06-29):
  - GPIO38: WS2812B LED控制器 ✅
  - RMT外设: 10MHz分辨率，精确时序控制 ✅
  - 系统状态指示: 完整的启动和运行状态监控 ✅
  - 音频问题诊断: 实时音频流监测和警告 ✅

### ⚠️ 配置约束
- **仅支持音量控制按键** - 不要添加其他按键类型 (如主按键、紧急停止等)
- **GPIO配置错误** 会导致 "GPIO_PIN mask error" - 确保引脚定义正确
- **I2S与I2C不能冲突** - 音频系统占用的GPIO不可复用
- **GPIO38专用于LED控制** - 不可复用为其他功能，RMT外设独占使用
- **双芯片分工明确** - ES7210专门录音，ES8311专门播放，不可混用功能
- **共享I2C总线** - 两个音频芯片使用相同的I2C总线，但地址不同
- **时钟信号共享** - MCLK/SCLK/LRCK被两个芯片同时使用，需要正确配置时序

### 🔧 故障排除记录
- **问题**: ESP_ERR_INVALID_ARG + GPIO_PIN mask error
- **原因**: 引用了不存在的按键GPIO定义
- **解决**: 移除定义
- **修复日期**: 2024-06-29

- **问题**: LED诊断系统实现
- **需求**: 实时系统状态指示和音频问题诊断
- **实现**: GPIO38 + WS2812B + RMT外设控制
- **功能**: 启动状态、运行监控、错误诊断、音频流检测
- **完成日期**: 2024-06-29

### 🔗 相关文件
- 按键处理: `components/audio_processor/button_handler.c`
- LED控制器: `main/led_controller.h` 和 `main/led_controller.c`
- 音频流配置: `components/audio_processor/audio_stream_custom_toy.c`
- 测试指南: `BUTTON_TEST_GUIDE.md`, `SPEAKER_TEST_GUIDE.md`

## 🎵 双芯片音频架构链路说明

### 录音链路 (ES7210 ADC专用)
```
麦克风 → MIC_BIAS → ES7210(ADC) → I2S(SDOUT) → ESP32-S3 → RTC引擎
```
- **麦克风偏置**: ES7210提供MIC_BIAS电压给电容麦克风
- **信号调理**: ES7210内置前置放大器和滤波器
- **数字输出**: 经I2S接口输出到ESP32-S3

### 播放链路 (ES8311 DAC专用)  
```
ESP32-S3 → I2S(DSDIN) → ES8311(DAC) → DAC_OUTP/OUTN → NS4150B功放 → 扬声器
```
- **数字输入**: ESP32-S3通过I2S发送播放数据到ES8311
- **DAC转换**: ES8311将数字音频转换为模拟信号
- **差分输出**: DAC_OUTP/OUTN差分信号驱动NS4150B功放

### 控制链路 (共享I2C总线)
```
ESP32-S3 → I2C → ES7210配置寄存器 → 录音增益/滤波器控制
                     ↓
                ES8311配置寄存器 → 播放音量/EQ控制
```

### 双芯片协同工作
- **时钟同步**: 两芯片共享MCLK、SCLK、LRCK时钟信号
- **独立控制**: 分别通过I2C配置各自的参数
- **可能的消噪连接**: ES7210的参考信号可能连接到ES8311用于回声消除

## 🚦 LED诊断系统 (GPIO38)

### 硬件配置
- **LED类型**: WS2812B 可编程RGB LED
- **GPIO引脚**: GPIO38
- **控制方式**: ESP32-S3 RMT (Remote Control Transceiver) 外设
- **时钟分辨率**: 10MHz (0.1μs精度)
- **数据格式**: GRB (绿-红-蓝) 24位颜色

### 🎨 LED状态指示系统

#### **系统启动阶段**
| LED颜色 | 状态 | 描述 |
|---------|------|------|
| 🤍 **白色** | LED初始化 | WS2812B控制器初始化成功 |
| 🔵 **蓝色** | 基础系统就绪 | 外设、WiFi、音频硬件初始化完成 |
| 🟡 **黄色** | RTC初始化中 | 正在初始化Volc RTC引擎 |
| 🟣 **紫色** | RTC就绪 | RTC引擎初始化成功，音频管道建立 |
| 🟢 **绿色** | 系统就绪 | 系统完全启动，可以正常使用 |

#### **运行状态监控**
| LED颜色 | 状态 | 描述 |
|---------|------|------|
| 🟢 **绿色** | 正常运行 | 系统正常，音频流正常 |
| 🌟 **青色** | 唤醒检测 | 检测到唤醒词 "嗨，乐鑫" |
| 🟣 **紫色** | 音频接收 | 正在接收远程音频数据 |
| 🔴 **红色** | 音频故障 | 麦克风问题或音频流异常 |

#### **错误诊断**
| LED颜色 | 问题类型 | 可能原因 | 解决方案 |
|---------|----------|----------|----------|
| 🔴 **红色** | RTC初始化失败 | 网络连接问题、认证失败 | 检查WiFi、检查配置参数 |
| 🔴 **红色** | 音频问题 | 麦克风硬件故障、95%以上静音 | 检查麦克风连接、ES7210增益设置 |
| **停留在某颜色** | 系统卡住 | 特定模块初始化失败 | 根据停留颜色确定故障模块 |

### 📊 音频诊断功能
- **静音检测**: 自动检测音频流中95%以上静音情况
- **音频统计**: 每5秒输出音频活动统计信息
- **自动恢复**: 音频问题解决后自动恢复绿色状态
- **远程音频**: 接收远程音频时闪烁紫色

### 🔧 技术实现
```c
// LED控制API
esp_err_t led_controller_init(void);           // 初始化LED控制器
esp_err_t led_set_color(rgb_color_t color);    // 设置LED颜色
esp_err_t led_breathing_effect(...);           // 呼吸效果
esp_err_t led_stop_effects(void);              // 停止效果

// 预定义颜色
RGB_COLOR_RED, RGB_COLOR_GREEN, RGB_COLOR_BLUE,
RGB_COLOR_WHITE, RGB_COLOR_YELLOW, RGB_COLOR_PURPLE,
RGB_COLOR_CYAN, RGB_COLOR_OFF
```

### 🚨 故障排除指南
1. **LED不亮**: 检查GPIO38连接，确认WS2812B供电
2. **LED颜色错误**: 检查GRB数据格式，验证RMT时序配置
3. **LED卡在某颜色**: 根据颜色对照表确定故障阶段
4. **频繁红色闪烁**: 音频硬件问题，检查麦克风和ES7210/ES8311双芯片系统

### ⚡ 性能参数
- **响应时间**: <10ms LED颜色切换
- **功耗**: ~60mA @ 全亮度白色
- **刷新率**: 根据系统事件实时更新
- **可靠性**: RMT硬件时序，抗干扰能力强

### 📝 维护日志
- **2024-06-29**: LED诊断系统设计并实现
- **2024-06-29**: 音频架构更正为ES7210(ADC) + ES8311(DAC)双芯片设计
- **已测试**: 基础颜色切换、系统状态指示
- **待测试**: 音频流监控、故障恢复机制
- **架构特点**: 分离式音频设计，录音和播放使用专用芯片，提供更好的音质