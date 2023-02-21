---
title: RobotMaster（校内赛）赛程记录
tags: []
id: '32'
categories:
  - - 电子信息
date: 2019-11-09 04:55:00
---

## 前言

因为我对硬件感兴趣，曾再高中划水摸鱼间了解过单片机。于是在比赛宣传（吐槽一下：其实我觉得主办方并没有过多的宣传，可能比较穷，毕竟开始报名的时候我身边的同学都不太了解这个比赛）的时候就和别班的同学一起组队了。这篇文章就是记录在准备赛事期间遇到的一些问题以及一些总结的经验。

因为比赛是面向全校的，所以大一的同学大部分必定都会吃亏。幸好我接触计科较早，在计科领域也认识了不少的dalao（首先感谢@Angelic47姐姐的帮助），所以在准备赛事的过程中也算比较顺利。毕竟为是初次接触单片机，就选了个最简单的题目：避障寻迹小车

## 赛程记录

### 复赛阶段

复赛于11.24号 下午13:00开始。在距比赛还有40分钟开始时我们就赶到了场地，在调试的时候虽然表现比大部分场上的人来说较好，但对我来说依然不尽人意。

于是尽管赛事迫在眉睫，我们依然在不停地测试小车，不停地改程序。幸运的是轮到我们正式比赛的时候小车车的表现比我们预期的好很多，几乎是最完美的一次。

最终成绩排到第二名，第一名是学姐们的车。取得这个成绩也已经很满足了，接下来就是准备决赛。在这里先立个Flag，我们要冲着第一名去･ω･)ﾉ

## 超声波避障模块

选购超声波模块的时候种类太多，就在淘宝马虎的看了一下参数，然后挑了个看起来性价比超高，牛拉吧唧的模块 HY-SRF05（也确实性价比比较高 性能稳定，测度距离精确。能和国外的SRF05,SRF02等超声波测距模相媲美。模块高精度，盲区（2cm）超近）

模块长这样：  
（以后补图）

**0.原理解释**

*   波发射器向某一方向发射超声波，在发射的同时开始计时，超声波在空气中传播，途中碰到障碍物就立即返回来，超声波接收器收到反射波就立即停止计时。根据**S=V\*t**就可以计算出被测物体离模块的距离。
*   公式推导：声音在20°C，空气中传播的速度为343m/s。换算一下单位为**0.0343cm/μs**。  
    设声波从发射到被接收的时间为**T**。则声波从发射到撞到被测物体所花费的时间为**T/2**。则距离公式为**S=0.01715\*T**

**1\. 丝印解释**

*   _ECHO_：接收返回的信号，如果有信号，ECHO引脚便会输出高电平。高电平持续的时间就是超声波从发射到返回的时间。
*   _TRIG_: 给TRIG引脚高于10μs的高电平信号时SR04模块测距功能便会被触发。
*   _GND_: 接地引脚
*   _VCC_: 接5V电源

**2\. 线路连接**

**3\. 程序编写**

```
float Qifd(int TrigPin,int EchoPin,int interval)
//超声波测距 返回一个float型数据。单位为cm
{
float distance;
int pulse_;

pinMode(TrigPin, OUTPUT);
// 要检测引脚上输入的脉冲宽度，需要先设置为输入状态
pinMode(EchoPin, INPUT);

// 产生一个10us的高脉冲去触发TrigPin
digitalWrite(TrigPin, HIGH);
delayMicroseconds(10);
digitalWrite(TrigPin, LOW);
// 检测脉冲宽度，并计算出距离
pulse_=pulseIn(EchoPin, HIGH);
distance = 0.01715*pulse_;
return distance;

delay(1000);
}
```

> 参考资料：[https://blog.csdn.net/qq\_31077649/article/details/72581968](https://blog.csdn.net/qq_31077649/article/details/72581968)