# EI209 Computer-Organization-Final-Project
【2019-2020Spring-EI209】 计算机组成 Computer Organization Final Project王正518021910079
2020 年5 月12 日

目录

1 实验要求1
2 思路、困难及解决方法1
2.1 整体框架及中断函数. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 1
2.2 第一问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 2
2.3 第二问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 2
2.4 第三问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 2
2.5 第四问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 2
2.6 其他问题. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 3
3 结果和现象3
3.1 第一问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 3
3.2 第二问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 3
3.3 第三问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 3
3.4 第四问. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 4
4 总结4

## 1 实验要求

1. 展示当前的日期。如：今天是4 月28 日，则需要在数码管上显示“0428”

2. 展示当前的时间。如：现在是14:15，则需要在数码管上显示“1415”。你可以设定任意的
    时间初始值，但时间需要每过1 分钟变化1 次(这里的1 分钟指仿真软件中的1 分钟)。

3. 实现1000 秒的倒计时。即初始值为1000，每过1 秒数字变化1 次。

4. 通过BT2 按钮实现上述1）-3）中显示内容的切换

## 2 思路、困难及解决方法

###   2.1 整体框架及中断函数

  通过8253 的Timer0，令每毫秒执行一次中断服务函数，每执行一次中断服务程序，Timer-
  CountDisplay 和msec 均自加1，当msec=1000 时，令sec 和_sec 加1，同时置零msec。
  1
  在主函数中，程序通过一个无限循环来不断检查变量TimerCountDisplay 的数值。当
  DisplayCount 等于5 时，说明已经产生了5 次中断，时间已过去了5ms。这时程序进入到模
  式选择（CheckMode）部分，选中模式后，跳转到相应的模式下运行代码，在数码管上显示需
  要显示的内容。

###   2.2 第一问

  第一问较为简单，在选入Mode1 后，每5ms 刷新一位数码管，DisplayIndex 表示数码
  管位数0,1,2,3，DisplayVal 表示数码管显示内容；我们通过CMP 指令来判断数码管位数，
  若不符合则跳转到下一位，选中数码管位数后赋值DisplayVal，之后调用PLAY 函数来控制
  L8255PA 和L8255PB 使内容显示在数码管的制定位上。
  因此我们分别为每一位赋值0,5,1,3，即可显示出设定的初始日期“0513”

###   2.3 第二问

  选入Mode2 后，首先进行分钟数min，小时数hour 的计算（setmin，sethour）。根据
  中断服务程序中得到的sec 值，当sec=60 时，令min+1，同时sec 置零；当min=60 时，
  令hour+1，同时置零min 的值。这样就得到了分钟数和小时数。值得注意的是，当小时数
  hour=24 时，要将hour 置零。这一操作在setday 中实现。
  得到hour 和min 后，和第一问一样，要逐位显示。第0 位和第1 位显示小时数：在第
  0 位，对hour 进行除法操作，令hour 除以10，所得的商为hour 十位上的数字，所得的余
  数入栈，在第1 位时出栈并显示，即为hour 个位上的数字。
  第2 位和第3 位显示小时数：在第2 位，对min 进行除法操作，令min 除以10，所得
  的商为min 十位上的数字，所得的余数入栈，在第3 位时出栈并显示，即为min 个位上的数
  字。

###   2.4 第三问

  选入Mode3 后，根据总秒数_sec（从程序开始运行以来经过的总秒数），得到1000-_，
  该值即为要在数码管显示的数值。在第0 位将该值除以1000，商为该数值的千位，赋值给
  DisplayVal，将余数入栈；到第1 位，令余数出栈，成为新的被除数，除以100，则可得到数
  值的百位，赋值给DisplayVal，余数依旧入栈；到第2 位，余数出栈，除以10，得到数值的
  十位，赋值给DisplayVal，余数入栈，在第3 位出栈，赋值给DisplayVal，即数值的个位。
  综上，通过除法操作，将倒计时数值逐位显示在数码管上。

###   2.5 第四问

  第4 问对我来说是难度最大的疑问，也是调试和修改时间最长的一问。一开始我的思路
  是利用74LS74 连接8255 的PortC，点击按钮改变PC0 的状态，同时通过改变PC4 来对
  74LS74 来进行reset（如图1），然后通过每0.1s 检测PortC 来实现模式的转换。但是在程序
  运行时一直出错，在按下第一次按钮后PC0 就一直保持高电平状态无法复位，而我也没有检
  查出代码和连线的错误。
  在一直debug 无果后，我采用了另一种思路：直接将BT2 接在PC0 上，另一端接地（如
  图2）。每按一次按钮PC0 被置1，每0.1s 检测PC0 状态，若PC0=1 则Mode+1，当然还
  要利用CMP 指令保证Mode 不可以超过3。这个简单粗暴的方法没有使用74LS74，但实现
  了功能切换的效果。
  2
  图1: 思路一的连线
  图2: 思路二的连线

###   2.6 其他问题

  一开始数码管的闪烁问题比较严重，于是我先调用了delay 函数来解决，让每一位显示
  有一定的显示延时。然后我修改了CPU 的频率，将原先的500K 修改为1M，闪烁现象大有
  改观。然后我又优化、精简了部分代码，减少了一些冗余的操作，尽量减少乘除而多用加减，
  以减少代码运行时间。最后，闪烁问题在很大程度上被解决。

##   3 结果和现象

###   3.1 第一问

  数码管显示预设日期“0513”（图3）
  图3: 日期显示“0513”

###   3.2 第二问

  数码管显示初始时间“2359”，并在仿真时间一分钟后同步更新时间（图4）。（偶尔显示
  更新会有几秒钟的延迟）

###   3.3 第三问

  数码管显示1000s 倒计时，每过1s 显示数字减一（图5）。
  3
  图4: 一分钟后由“2359”变为“0000”
  图5: 倒计时1000s

###   3.4 第四问

  每按一次按钮，述1）-3）中显示内容，具体结果见视频。

##   4 总结

  这次的final project 让我学习和巩固了很多知识，如Proteus 软件的使用，中断服务程序
  的使用，8255,8253 和七段数码管的工作原理，能够熟练使用一些之前并不太了解的指令，如
  DIV，CMP，JNZ，大大加强了使用汇编语言的熟练程度。
  4
