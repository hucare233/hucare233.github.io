---
sort: 3
---

# 驱动器

## DJESC

---

## ELMO

---

## VESC

[![VESC官网](https://img.shields.io/badge/-VESC官网-red)](http://vesc-project.com)
[![VESC_GitHub](https://img.shields.io/badge/-VESC_Github-blue)](http://github.com/vedderb)
[![VESC论坛](https://img.shields.io/badge/-VESC论坛-green)](http://vesc-project.com)

VESC电调是由Benjamin Vedder研究并开源的一款强大的电机电子调速器。电流最高可达100+A（具体多少忘了），但电流过高会烧驱动器，有过先例！！所以最高一般80A足以，远高于以前的ELMO驱动器。  
VESC常用于速度控制场合，如航模，滑板车电机控制，电调带有三环控制，但其自己的位置控制只允许在360°范围内进行旋转，多有不便。如想实现常用位置控制需在代码层进行实现。  
VESC电调有诸多款，当前队内购买的主要为VESC6。

### ㈠上位机

BLDC Tool VESC 电调调试工具是一款运行在windows下的本杰明电调的调试工具，在对VESC开源电调进行调试时需要使用到这个程序，支持BLDC 和FOC，支持更新固件，需要使用COM串口（安卓线）进行连接后再调试。  
目前队内常用的上位机版本为vesc_tool_0.95，该版本上位机较老，功能较少，但可满足当前的需求。该版本上位机操作界面如图所示：
![VESC_main_pic](../md_pictures/VESC_main_pic.png)

### ㈡上位机操作

>初学者请先看完 *(学习资料\机构\VESC驱动器\VESC工具\0.VESC工具手册_ VESC项目.pdf等6个文件)*

了解以下界面的功能操作

1. Wizards 向导界面

**注意事项:**  
```
1.需要等到本杰明板初始化完成（灯亮再灭）才可正常连接，有时J-link和USB一同插在集线器上会无法连接，这时候需先拔掉J-link待连接成功后再重新插入。
2.电机上电时，若该电机无法通过方向键正常运转，但已有过整定参数，需注意应先给板子刷一遍默认参数，再将mc.xml下载至本杰明板，之后在FOC->Encoder界面再检测一次编码器就好。
3.电机三相电插在本杰明电源输出口的时候之前并管过线序，它只会影响同一电流输入后的电机转向。
但编码器的正转方向是固定的，本杰明固件内的程序电流输出方式也是固定的。
以电机输出轴向外，编码器向自己，此时电机逆时针旋转编码器为正向。需要检测电流方向与编码器方向是否同向，若非，则于Motor Settings -> General -> General -> Invert Motor Direction进行更改。否则PID会出错。
```

2. Motor Settings 电机设置界面

   - General


3. App Settings 上位机设置界面

    - General  
    此处配置间

4. Data Analysis 数据分析界面

### ㈢固件库

[![固件库](https://img.shields.io/badge/-固件库-important)](http://vesc-project.com)

#### 1.所做修改

我们使用的是VESC固件库keil移植版（淘宝赠送）。根据我们的需求对固件库的一些参数进行了修改：  
①占空比反馈改为了位置反馈：--反馈数据为 绝对角度*10，变量类型u16  
②变量修改
|变量|初值|更改后|所在文件|
|:-:|:-:|:-:|:-:|
|MCCONF_S_PID_MIN_RPM|900.0f|0.0f|mcconf_default.h|
|APPCONF_CONTROLLER_ID|0|1|appconf_default.h|
|APPCONF_SEND_CAN_STAUS|faule|true|appconf_default.h|
|APPCONF_SEND_CAN_STATUS_RATE_HZ|100|2000|appconf_default.h|
|APPCONF_CAN_BAUD_RATE|100|2000|appconf_default.h|

#### 2.代码分析

+ appconf_default

上位机配置文件，其内参数对应上位机App Settings

+ comm_can [ CAN通信文件 ]

`static THD_FUNCTION(cancom_process_thread, arg)`函数是VESC接收报文处理函数，即我们发送的报文。CAN命令详见`CAN_PACKET_ID`枚举结构。发送至VESC的报文通信格式须与该文件内定义的格式一致。  
原固件库反馈为rpm &  current & duty;将duty改为pos方便进行位置计算。此处位置为相对角度，仍需累加计算。  
VESC运行过程中，需持续给其发送报文，否则电调将会断连。

+ digital_filter [ 数字滤波器 ]

包含常见的几种数字滤波器，可以学习一下。

+ mcconf_default [ 电机配置文件 ]

其内参数对应上位机Motor Settings

+ mcpwm_foc [ foc算法 ]

+ utils [ 数学函数 ]

该文件是一个小型的函数集，内含诸多有用的数学工具，建议移植到自己的程序里面。

### ㈣程序

>详见vesc.c & vesc.h

#### 1.电流环

1. 防止实现延时，将电流环报文直接发送，不要放在队列里面

#### 2.速度环

1. 目前我的VESC_RPM_mode_I()里面用的本杰明的PID算法，只是系数变了，它的系数是在是太小了。

#### 3.位置环

1. 防止丢失位置信息，将报文反馈频率设置为2000hz

---

## EPOS

---

## ODRIVE
