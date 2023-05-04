- 👋 Hi, I’m @chencaiqian
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
chencaiqian/chencaiqian is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->




//连续驱动.2-2.增加Web 开关代码，增加 按键开关代码。
//esp8266mcu D1,D2.D3.D4.连IN1.IN2.IN3.IN4。
//电压5V   电流0.1A
//验证成功:用D5----> GND按键。web网页控制‘正反转’
#include <AccelStepper.h>
#include <ESP8266WiFi.h>
#include <ESPAsyncTCP.h>
#include <ESPAsyncWebServer.h>

// Wi-Fi SSID 和密码
const char* ssid = "ASUS_F3_2G";
const char* password = "7758258dy";

// 创建异步Web服务器实例，监听80端口
AsyncWebServer server(80);

// 定义4个引脚分别连接IN1, IN2, IN3, IN4
#define IN1 D1
#define IN2 D2
#define IN3 D3
#define IN4 D4

// 定义按键引脚
#define BUTTON_PIN D5

// 创建AccelStepper实例，设置类型为FULL4WIRE，连接方式为驱动器及步进电机共用电源
AccelStepper stepper(AccelStepper::FULL4WIRE, IN1, IN2, IN3, IN4);

bool isForward = true; // 定义控制步进电机方向的变量，默认为正转

void setup() {
  // 连接Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  
  Serial.println("WiFi connected");
  Serial.print("Local IP: ");
  Serial.println(WiFi.localIP());
  
  // 将4个引脚设为输出模式
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  
  // 设置按键引脚为输入模式
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  
  // 启动异步Web服务器
  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request) {
    String html = "<html><head><title>Step Motor Control</title></head><body>";
    html += "<h1>Step Motor Control</h1>";
    
    // 生成一个表单，包含一个按钮和一个隐藏的输入框
    html += "<form method=\"POST\" action=\"/reverse\"><input type=\"hidden\" name=\"reverse\">";
    html += "<button type=\"submit\">Reverse</button>";
    html += "</form>";
    
    html += "<p>Current direction: " + String(isForward ? "forward" : "backward") + "</p>";
    html += "</body></html>";
    request->send(200, "text/html", html);
  });
  
  server.on("/reverse", HTTP_POST, [](AsyncWebServerRequest *request) {
    // 反转isForward变量的值
    reverse();
    request->redirect("/");
  });
  
  server.begin();
}

void loop() {
  // 如果按键按下，则反转方向
  if (digitalRead(BUTTON_PIN) == LOW) {
    reverse();
  }
  // 根据isForward变量来决定步进电机的运动方向
  if (isForward) {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  } else {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
  }
  
  delay(2);
 
  if (isForward) {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
  } else {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  }
  
  delay(2);
 
  if (isForward) {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, HIGH);
  } else {
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  }
  
  delay(2);
 
  if (isForward) {
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  } else {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, HIGH);
  }
  
  delay(2);
}

void reverse() {
  // 反转isForward变量的值
  isForward = !isForward;
  // 根据新的方向设置电机引脚的电平状态
  if (isForward) {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  } else {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
  }
}
