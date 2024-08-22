# jianing PTC加热模块
## 需求逻辑
按下按钮，开启加热模块。松开按钮，加热模块停止工作

## 实现原理
PTC加热模块：随着温度升高，电阻值逐渐增加（用来控制温度） 低温下，PTC加热模块的电阻较低，因此通过的电流较大，模块迅速升温。当达到某一设定温度时，电阻显著增大，电流减小，模块维持在该温度附近，不再显著升温。   


按钮：开关在没有被按下时，两个引脚之间是不导通的，只有在按下时，两个引脚才导通。    

## 代码

```C++

const int buttonPin = 2; // 定义 数字2 代表按钮
const int ptcPin = 3;    // 定义 数字3 代表加热模块

void setup() { //初始化内容
  // 初始化串口通讯
  Serial.begin(9600);
  
  // 把按钮设置为为输入模式
  pinMode(buttonPin, INPUT_PULLUP);
  
  // 加热模块设置为输出
  pinMode(ptcPin, OUTPUT);

  // 初始化时关闭加热模块
  digitalWrite(ptcPin, LOW);

  Serial.println("系统初始化完成");
}

void loop() { //在代码运行过程中不停循环重复运行
  // 读取按钮状态
  int buttonState = digitalRead(buttonPin);
  
  // 打印按钮状态
  Serial.print("按钮状态: ");
  //如果按钮被按下
  Serial.println(buttonState == HIGH ? "按下" : "松开");
  
  // 如果按钮被按下
  if (buttonState == HIGH) {
    // 打开加热模块
    digitalWrite(ptcPin, HIGH);
    Serial.println("加热模块状态: 打开");
  } else {
    // 关闭 PTC 模块
    digitalWrite(ptcPin, LOW);
    Serial.println("PTC 模块状态: 关闭");
  }

  // 延时1秒，避免串口输出过于频繁
  delay(1000);
}

```
## 检测按钮是长按还是短按
```C++
/*

 * 由 ArduinoGetStarted.com 创建
 *
 * 此示例代码属于公共领域
 *
 * 教程页面: https://arduinogetstarted.com/tutorials/arduino-button-long-press-short-press
 */

// 常量值不会改变。在这里用来设置引脚编号:
const int BUTTON_PIN = 7; // 按钮引脚号
const int LONG_PRESS_TIME  = 1000; // 1000 毫秒，定义长按的时间阈值

// 变量值会改变:
int lastState = LOW;  // 输入引脚的前一个状态
int currentState;     // 输入引脚的当前状态
unsigned long pressedTime  = 0;   // 按钮按下时的时间
unsigned long releasedTime = 0;   // 按钮释放时的时间

void setup() {
  Serial.begin(9600);                // 初始化串口通信
  pinMode(BUTTON_PIN, INPUT_PULLUP); // 设置按钮引脚为输入模式，并启用内部上拉电阻
}

void loop() {
  // 读取按钮的状态:
  currentState = digitalRead(BUTTON_PIN);

  if(lastState == HIGH && currentState == LOW) {      // 按钮被按下
    pressedTime = millis();
  } 
  else if(lastState == LOW && currentState == HIGH) { // 按钮被释放
    releasedTime = millis();

    long pressDuration = releasedTime - pressedTime;

    if(pressDuration > LONG_PRESS_TIME) {
      Serial.println("检测到长按");
    } 
    else {
      Serial.println("检测到短按");
    }
  }

  // 保存上一次的状态
  lastState = currentState;
}
```
## 接线
接线方式（适用于带有放大器的扬声器模块）：
VCC（电源）：连接到Arduino的5V电源引脚。
GND（地）：连接到Arduino的GND引脚。
音频输入：连接到HX6210T模块的SPK+和SPK-引脚。
具体接线步骤：
假设你使用的是HX6210T MP3模块和一个带有放大器的扬声器模块：

HX6210T与Arduino接线：

VCC: 连接到Arduino的5V电源。
GND: 连接到Arduino的GND。
TX: 连接到Arduino的RX引脚（例如，D10）。
RX: 连接到Arduino的TX引脚（例如，D11）。
HX6210T与扬声器接线：

SPK+: 连接到扬声器模块的音频输入正极（通常标有“IN+”或“A+”）。
SPK-: 连接到扬声器模块的音频输入负极（通常标有“IN-”或“A-”）。
扬声器模块与电源接线：

VCC: 连接到Arduino的5V电源（如果模块需要不同的电压，请确保使用合适的电源）。
GND: 连接到Arduino的GND。

## 录音 储存声音 播放声音
```C++
#include <SoftwareSerial.h>

// 常量值不会改变。在这里用来设置引脚编号:
const int BUTTON_PIN = 7;           // 按钮引脚号
const int LONG_PRESS_TIME = 1000;   // 1000 毫秒，定义长按的时间阈值

// 变量值会改变:
int lastState = HIGH;               // 初始化为高电平，因为使用上拉电阻
int currentState;                   // 当前按钮状态
unsigned long pressedTime = 0;      // 按钮按下时的时间
unsigned long releasedTime = 0;     // 按钮释放时的时间

SoftwareSerial mySerial(10, 11);    // RX=10, TX=11，可以根据实际接线调整

void setup() {
  Serial.begin(9600);                // 初始化串口通信
  pinMode(BUTTON_PIN, INPUT_PULLUP); // 设置按钮引脚为输入模式，并启用内部上拉电阻
  
  // 读取初始状态，防止在启动时误判按钮状态
  lastState = digitalRead(BUTTON_PIN);

  mySerial.begin(9600);              // 初始化与HX6210T模块的通信
}

void loop() {
  // 读取按钮的状态:
  currentState = digitalRead(BUTTON_PIN);

  if (lastState == HIGH && currentState == LOW) { // 按钮被按下
    pressedTime = millis();
  } 
  else if (lastState == LOW && currentState == HIGH) { // 按钮被释放
    releasedTime = millis();

    long pressDuration = releasedTime - pressedTime;

    if (pressDuration > LONG_PRESS_TIME) {
      Serial.println("检测到长按");
      delay(1000);           // 延迟一秒再播放
      playBeep();            // 播放“滴”声
    } 
    else if (pressDuration > 50) { // 添加防抖机制，忽略极短的误触发
      Serial.println("检测到短按");
      delay(1000);           // 延迟一秒再播放
      playFakeRecording();   // 播放假装的录音
    }
  }

  // 保存上一次的状态
  lastState = currentState;
}

void playBeep() {
  Serial.println("播放滴声...");
  mySerial.write(0x7E);  // 开始命令头
  mySerial.write(0x02);  // 数据长度
  mySerial.write(0x01);  // 播放“滴”声文件 (0001.mp3)
  mySerial.write(0xEF);  // 结束命令
  delay(1000);           // 假设播放时长1秒
  Serial.println("滴声播放结束");
}

void playFakeRecording() {
  Serial.println("播放假装的录音...");
  mySerial.write(0x7E);  // 开始命令头
  mySerial.write(0x02);  // 数据长度
  mySerial.write(0x02);  // 播放“录音”文件 (0002.mp3)
  mySerial.write(0xEF);  // 结束命令
  delay(5000);           // 假设播放时长5秒
  Serial.println("假装录音播放结束");
}

```

## 最终代码
```C++
#include <SoftwareSerial.h>

// 常量值不会改变。在这里用来设置引脚编号:
const int BUTTON_PIN = 2;           // 按钮引脚号
const int PTC_PIN = 3;              // 加热模块引脚号
const int LONG_PRESS_TIME = 1000;   // 1000 毫秒，定义长按的时间阈值

// 变量值会改变:
int lastState = HIGH;               // 初始化为高电平，因为使用上拉电阻
int currentState;                   // 当前按钮状态
unsigned long pressedTime = 0;      // 按钮按下时的时间
unsigned long releasedTime = 0;     // 按钮释放时的时间

SoftwareSerial mySerial(10, 11);    // RX=10, TX=11，可以根据实际接线调整

void setup() {
  Serial.begin(9600);                // 初始化串口通信
  pinMode(BUTTON_PIN, INPUT_PULLUP); // 设置按钮引脚为输入模式，并启用内部上拉电阻
  pinMode(PTC_PIN, OUTPUT);          // 设置PTC引脚为输出模式
  
  // 初始化时关闭加热模块
  digitalWrite(PTC_PIN, LOW);

  // 读取初始状态，防止在启动时误判按钮状态
  lastState = digitalRead(BUTTON_PIN);

  mySerial.begin(9600);              // 初始化与HX6210T模块的通信

  Serial.println("系统初始化完成");
}

void loop() {
  // 读取按钮的状态:
  currentState = digitalRead(BUTTON_PIN);

  if (lastState == HIGH && currentState == LOW) { // 按钮被按下
    pressedTime = millis();
  } 
  else if (lastState == LOW && currentState == HIGH) { // 按钮被释放
    releasedTime = millis();

    long pressDuration = releasedTime - pressedTime;

    if (pressDuration > LONG_PRESS_TIME) {
      Serial.println("检测到长按");
      delay(1000);           // 延迟一秒再播放
      playBeep();            // 播放“滴”声
    } 
    else if (pressDuration > 50) { // 添加防抖机制，忽略极短的误触发
      Serial.println("检测到短按");
      delay(1000);           // 延迟一秒再播放
      triggerPTC();          // 触发PTC模块
      playFakeRecording();   // 播放假装的录音
    }
  }

  // 保存上一次的状态
  lastState = currentState;
}

void playBeep() {
  Serial.println("播放滴声...");
  mySerial.write(0x7E);  // 开始命令头
  mySerial.write(0x02);  // 数据长度
  mySerial.write(0x01);  // 播放“滴”声文件 (0001.mp3)
  mySerial.write(0xEF);  // 结束命令
  delay(1000);           // 假设播放时长1秒
  Serial.println("滴声播放结束");
}

void playFakeRecording() {
  Serial.println("播放假装的录音...");
  mySerial.write(0x7E);  // 开始命令头
  mySerial.write(0x02);  // 数据长度
  mySerial.write(0x02);  // 播放“录音”文件 (0002.mp3)
  mySerial.write(0xEF);  // 结束命令
  delay(5000);           // 假设播放时长5秒
  Serial.println("假装录音播放结束");
}

void triggerPTC() {
  // 打开加热模块
  digitalWrite(PTC_PIN, HIGH);
  Serial.println("加热模块状态: 打开");
  
  // 在此版本中，PTC模块将保持开启状态
}
```
