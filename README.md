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

## 录音 储存声音 播放声音
```C++
#include <SoftwareSerial.h>
#include <DFRobotDFPlayerMini.h>

const int buttonPin = 2;   // 按钮引脚
const int micPin = A0;     // 麦克风引脚

SoftwareSerial mySoftwareSerial(10, 11); // RX, TX
DFRobotDFPlayerMini myDFPlayer;

bool isPlaying = false;

void setup() {
  pinMode(buttonPin, INPUT_PULLUP);
  pinMode(micPin, INPUT);
  Serial.begin(9600);
  mySoftwareSerial.begin(9600);

  if (!myDFPlayer.begin(mySoftwareSerial)) {
    Serial.println("无法初始化MP3-TF-16P模块");
    while (true);
  }
  
  myDFPlayer.setTimeOut(500); // 设定超时时间
  myDFPlayer.volume(10); // 设置音量（0~30）
  myDFPlayer.EQ(DFPLAYER_EQ_NORMAL); // 设置EQ模式

  Serial.println("MP3-TF-16P模块初始化成功");
}

void loop() {
  if (digitalRead(buttonPin) == LOW) {
    delay(50); // 防抖
    if (isPlaying) {
      stopPlaying();
    } else {
      startPlaying();
    }
    while (digitalRead(buttonPin) == LOW); // 等待按钮释放
  }
}

void startPlaying() {
  Serial.println("开始播放录音");
  isPlaying = true;
  myDFPlayer.play(1); // 播放
}

void stopPlaying() {
  Serial.println("停止播放");
  myDFPlayer.stop();
  isPlaying = false;
}
```

