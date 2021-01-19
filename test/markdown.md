---
sort: 8
---

# 驱动器注意事项

> 大疆队里用的有两款电机，分别是DJ3508和DJ2006,配套使用电调c620和c610，电机本身均配备编码器， 反馈数据见说明书.，新手友好型
>
> elmo驱动电机往往需要外加编码器，使用canopen协议，要求熟练使用上位机，会整定参数，
>
> vesc驱动器能够通过大电流，有望作为底盘电机驱动器使用，但是还未开发完全
>
> epos驱动器分24v和48v供电，f90电机配套，但是速度跟随性能不好，开发中

# Header 1-`DJ注意事项`

+ DJ电机电流是一起发的，包括所有电机。所以多个板子控制电机的时候不要将can2线连在一起
+ DJ电机开始位置记录因为在`pulse_caculate()`函数里面开始distance会先运行一次才会在对`valuePrv`进行赋值，所以会有`0~8192`的脉冲误差（已尝试解决一部分误差）
+ 将电机使能放在了start里面以达成延时使能的要求从而避免初始位置误差  
+ 2006发送can报文的频率大概是`0.5-1.7ms`，一圈脉冲数是8192，那么按照1.5ms一条报文来算，`阈值设定为4100`允许的最大速度比较大，在19970rpm左右，如果速度过大，则由于阈值的原因将会无法判断电机的转向。所以对于2006来说，速度不要超过19500,否则角度偏差会比较大(之前写转向程序就出过角度丢失的现象)，另外3508转速低阈值基本来说随便给= =
+ 理论一个板子可以同时控制8个电机，实际上有一定风险，最好不要超过6个电调在同一条总线上
+ 队里有两个云台电机6025，一个pitch轴，一个yaw轴，两种轴电机pid参数不同，正反转pid貌似也有区别，目前没有需求所以没用，有兴趣可以自行研究

# Header 2 -`ELMO注意事项`

> 电机模式枚举体因ELMO的电流、速度、位置为固定1，2，5
>
> enable只是一个状态，用于观测，并不能真正`像3508实现使能
>
> 关于ElmoCANopen协议详见`文档9.1.1.1`
>
> `intrinsic.PULSE`直接乘4方便计算
>
> JV、SP的计算方法  ——    `速度*编码器线数*4/60=jv(sp)`,位置计算同理
>
> U10电机有时候会犯病，可能是因为没有霍尔的影响，目前能找到的解决方法只有重新上电，日后可以研究下霍尔的使用
>
> PVT模式数组的设置`不可保存`，得在程序中手动添加
>
> PVT数组 `QP[1]=0 QV[1]=0;`，MP[3]: 0-不循环 1-循环
>
> RM=0在MO=0时发送，PV.PT=1在MO=1之后发送，在下一个BG执行

# Header 3-`VESC相关事项`

```
VESC [Github](http://github.com/vedderb) 地址
VESC [论坛](http://vesc-project.com/vesc_tool) 地址
VESC需要定时赋值否则会自动释放
有感模式下在速度大于openloop erpm时会切换到无感模式，在上位机把openloop erpm改大
ERPM是电角度转速，=RPM*POLES极对数。还有一个是Minimum ERPM，小于这个转速电机就不运行了
速度模式发送erpm
电机三相电插在本杰明电源输出口的时候之前并管过线序，它只会影响同一电流输入后的电机转向，但编码器的正转方向是固定的，本杰明固件内的程序电流输出方式也是固定的
以电机输出轴向外，编码器向自己，此时电机逆时针旋转编码器为正向。需要检测电流方向与编码器方向是否同向，若非，则于Motor Settings -> General -> General -> Invert Motor Direction进行更改
位置模式目前pid用了固件的速度环电流环，位置环自己编写，速度跟随效果很好，锁点电机会有轻微震荡
遗留问题：ASK: 能不能自己写速度环? 我尝试效果都不是很理想，但原理是可行的
```

---
|变量|初值|更改后|所在文件|
|:-:|:-:|:-:|:-:|
|MCCONF_S_PID_MIN_RPM|900.f|0.0f|mcconf_defult.h|
|APPCONF_CONTROLLER_ID|0|1|appconf_defult.h|
|APPCONF_SEND_CAN_STAUS|false|true|appconf_defult.h|
|APPCONF_SEND_CAN_STAUS_RATE_HZ|100|2000|appconf_defult.h|
|APPCONF_CAN_BAUD_RATE|CAN_BAUD_500K|CAN_BAUD_1M|appconf_defult.h|
---

# Header 4-`EPOS注意事项`

1. EPOS的电机使能需要连续在控制字写入`0x06 0x07 0x0F`,使能一次后只需再发送0x0F就行
2. 速度模式王锟学长已经摸得很清楚了，具体可以直接看他总结
3. 位置模式有两种模式，类似于编码器的绝对式和增量式，可以借助代码实践理解
4. 官方提供的halt和quickstop的刹车效果不很理想，平时基本不用，速度模式下的锁位置通过直接更改为位置模式完成
4. 王锟学长之前遇到过4个电机用机构的代码没问题，与主控加入调试后会有驱动器报错现象，初步结论可能是因为主控发的某些报文它也能接收
4. 位置模式再到达目标后，若想再次运动必须`再发一次0x0F`
5. epos没有查询电流的功能，只能查询当前扭矩在除以一个系数算出电流（参数在flat90手册里头）
6. 启动HMM模式，控制字写入`0x1F`，有两个归零速度，一个较快用于寻找机械寻零，一个较慢用来到达索引脉冲
7. HMM模式method设置为`actual method` ，设置当前位置为零点，`offset position 和 homing position均给0`

---


![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

Large images should always scale down and fit in the content container.

![Branching](https://guides.github.com/activities/hello-world/branching.png)

```
This is the final element on the page and there should be no margin below this.
```
