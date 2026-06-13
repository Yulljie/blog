# Paste1

循迹小车是机器人制作中非常经典的项目！它的控制逻辑其实非常符合直觉，本质上就是一个**“输入（看路） \rightarrow 思考（判断） \rightarrow 输出（转向）”**的循环过程。
无论你使用的是 Arduino、树莓派还是 STM32，代码的骨架和思路都是完全通用的。
## 循迹小车代码的标准框架
下面以最常见的 **Arduino（配双路红外传感器）** 为例，为你展现它的标准代码框架。它主要由四个核心部分组成：
```cpp
// ==========================================
// 1. 引脚与变量定义区 (给器官命名)
// ==========================================
// 红外传感器引脚（左、右）
const int SENSOR_LEFT = 2;
const int SENSOR_RIGHT = 3;

// 电机控制引脚（假设使用L298N驱动模块，PWM引脚控制速度）
const int MOTOR_LEFT_A = 5;
const int MOTOR_LEFT_B = 6;
const int MOTOR_RIGHT_A = 9;
const int MOTOR_RIGHT_B = 10;

// ==========================================
// 2. setup() 初始化区 (只在开机时运行一次)
// ==========================================
void setup() {
  // 传感器作为输入端（接收地面信号）
    pinMode(SENSOR_LEFT, INPUT);
      pinMode(SENSOR_RIGHT, INPUT);
        
          // 电机引脚作为输出端（向电机送电）
            pinMode(MOTOR_LEFT_A, OUTPUT);
              pinMode(MOTOR_LEFT_B, OUTPUT);
                pinMode(MOTOR_RIGHT_A, OUTPUT);
                  pinMode(MOTOR_RIGHT_B, OUTPUT);
                  }

                  // ==========================================
                  // 3. loop() 主循环区 (大脑不停地思考)
                  // ==========================================
                  void loop() {
                    // 【第一步：看】读取传感器状态 (假设检测到黑线时返回 HIGH，没检测到返回 LOW)
                      int leftState = digitalRead(SENSOR_LEFT);
                        int rightState = digitalRead(SENSOR_RIGHT);
                          
                            // 【第二步：想】根据传感器状态，决定小车怎么走
                              if (leftState == LOW && rightState == LOW) {
                                  // 左右都没踩到黑线 -> 路线很正，放心直行
                                      moveForward();
                                        } 
                                          else if (leftState == HIGH && rightState == LOW) {
                                              // 左边踩到黑线 -> 说明车子偏右了，需要向左修正
                                                  turnLeft();
                                                    } 
                                                      else if (leftState == LOW && rightState == HIGH) {
                                                          // 右边踩到黑线 -> 说明车子偏左了，需要向右修正
                                                              turnRight();
                                                                } 
                                                                  else {
                                                                      // 左右都踩到黑线 -> 可能是遇到了十字路口或终点，选择停下
                                                                          stopCar();
                                                                            }
                                                                            }

                                                                            // ==========================================
                                                                            // 4. 动作封装区 (手脚听指挥执行)
                                                                            // ==========================================
                                                                            void moveForward() {
                                                                              analogWrite(MOTOR_LEFT_A, 150);  // 给左轮合适的PWM速度(0~255)
                                                                                digitalWrite(MOTOR_LEFT_B, LOW);
                                                                                  analogWrite(MOTOR_RIGHT_A, 150); // 给右轮相同的速度
                                                                                    digitalWrite(MOTOR_RIGHT_B, LOW);
                                                                                    }

                                                                                    void turnLeft() {
                                                                                      digitalWrite(MOTOR_LEFT_A, LOW); // 左轮不动（或后退）
                                                                                        digitalWrite(MOTOR_LEFT_B, LOW);
                                                                                          analogWrite(MOTOR_RIGHT_A, 150); // 右轮前进，车子自然向左转
                                                                                            digitalWrite(MOTOR_RIGHT_B, LOW);
                                                                                            }

                                                                                            void turnRight() {
                                                                                              analogWrite(MOTOR_LEFT_A, 150); // 左轮前进
                                                                                                digitalWrite(MOTOR_LEFT_B, LOW);
                                                                                                  digitalWrite(MOTOR_RIGHT_A, LOW); // 右轮不动
                                                                                                    digitalWrite(MOTOR_RIGHT_B, LOW);
                                                                                                    }

                                                                                                    void stopCar() {
                                                                                                      digitalWrite(MOTOR_LEFT_A, LOW);
                                                                                                        digitalWrite(MOTOR_LEFT_B, LOW);
                                                                                                          digitalWrite(MOTOR_RIGHT_A, LOW);
                                                                                                            digitalWrite(MOTOR_RIGHT_B, LOW);
                                                                                                            }

                                                                                                            ```
                                                                                                            ## 核心框架解析
                                                                                                            > 💡 **为什么要把代码写成这样？**
                                                                                                            > 
                                                                                                             * **解耦设计（模块化）**：注意到第 4 部分的 moveForward()、turnLeft() 了吗？把具体的电机控制命令打包成一个个“动作盒”。这样我们在第 3 部分写主逻辑时，只要调用动作名字就行了。如果以后你想让小车跑得更快，只需要去修改动作盒里的速度数字，而不用动逻辑代码。
                                                                                                              * **高频刷新**：loop() 函数是以微秒级的速度在疯狂循环的。小车每走一毫米，都在经历“看路、判断、调整”的过程，这就保证了它不会轻易脱轨。
                                                                                                              ## 进阶方向
                                                                                                              这个基础框架在面对直线和缓弯时效果很好。但如果你想让它跑得更稳、更快，通常还会有两个进阶方向：
                                                                                                               1. **增加传感器数量**：使用 4 路、5 路甚至 8 路红外线传感器（灰度传感器阵列）。传感器越多，小车对轨道的感知就越细腻。
                                                                                                                2. **引入 PID 算法**：基础框架的转向是“非黑即白”的硬转向，容易导致小车“摇头晃脑”。引入 PID 算法后，可以根据小车偏离轨道的距离，平滑地调节左右轮的速度差，实现完美的丝滑过弯。
                                                                                                                你目前准备使用几路传感器来做这个小车？打算选用哪种主控板（比如 Arduino 还是树莓派）呢？

