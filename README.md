# ideapad-720s-13IKB
我买的日版机，CPU是i5 8250u，其它参数可看鲁大师导出的配置单，所有文件来自作者dragonflylee的  https://github.com/dragonflylee/Yoga-730-hackintosh  我只改了layoutid为22，并去掉了原项目里我暂时用不到的opencore文件。

### 使用方法：
用黑果小兵提供的10.15.1镜像，刻录一个安装盘，用此处的EFI替换安装U盘里的EFI，然后安装系统。
系统安装完成后，用此处的EFI替换系统盘里的EFI,关机拔掉U盘，开机设置启动顺序从系统盘启动，即可脱离u盘启动。
然后config.plist里改声卡layoutid为22。

### 可用项目
1. 触摸板已驱动
2. intel power gadget有变频，电池电量显示正常，并调整系统设置去掉睡眠，怕出问题，不用时关机就好了
3. 亮度快捷键可调
4. 声卡正常
5. 显卡正常
6. 自带网卡蓝牙暂时不管，需更换才行，目前暂用的USB迷你网卡comfast cf-811ac，官方提供驱动安装包，点击安装即可。
7. 雷电口没设备暂时没设备测


### 发现问题
目前发现触摸板在闲置一段时间，如5分钟多点，就会不响应，像完全没驱动一样，必须重启才可以正常，可能需要自己定制DSDT(或许voodooi2c本身有bug)，但我不会，这里有这个教程，我看了几遍还不太懂，有需要的去看看吧，参考教程：https://www.penghubingzhou.cn/2019/01/06/VoodooI2C%20DSDT%20Edit/ 或官方说明  https://voodooi2c.github.io

### 其它说明
有问题需要讨论可去 http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1817262&highlight=720s  我只是看到讨论里用这个EFI可以,就用了,其它不懂。



### 20191119关于网卡更新
鉴于USB网卡占用宝贵的USB口，我最终还是买了dw1820a，但是系统就是识别不出来，无论win下还是mac下，开始以为是网卡问题，就又买了bcm94360cs2加仕和技术的m2转苹果网卡那个延长线，把电池拿下来去掉扬声器，可以很好的放入这个网卡，反正扬声器几乎不用，但是遇到同样的问题，依然无论win还是mac都识别不到，后来群里有人说换网卡前要先屏蔽wifi，我就先bios里屏蔽wifi进系统，然后再解开屏蔽进系统，居然成功识别出来了，bcm94360cs2直接免驱，可以把相关驱动删了(AirportBrcmFixup.kext、
BT4LEContinuityFixup.kext、BrcmBluetoothInjector.kext)，可能有些人需要，所以仓库里我没去除，另外，dw1820a应该也是一样需要先bios屏蔽才识别吧，拆装比较麻烦就没再试dw1820a了。

### 20191120关于触摸板bug修复

#### 分析问题
仔细读了官方文档 https://voodooi2c.github.io 发现官方文档写的还是很清晰的，终于有点明白了，然后对照着dragonflylee原来的EFI，发现作者没有修改DSDT貌似，只是热补丁修改SSDT，当然这两者具体关系我也不太懂。所以现在的问题是我要按voodooi2c官方文档要求修复一份DSDT放进去，然后我又想到这个作者FuckDoctors（ 他EFI地址 https://github.com/FuckDoctors/ideapad-720s-13IKB ） 在issue里说他的DSDT只修改了触摸板和调节亮度的部分，就想可以参照着他的改。

#### 修复步骤
大致步骤：我把FuckDoctors的DSDT.dsl下载下来搜触摸板bios设备名：TPD0，然后找到他所有为voodooi2c打补丁的地方（作者用zhbchwin注释了这里地方），把需要的代码挪到我自己提取的本机的DSDT里，然后编译导出aml，放到我patch里，然后就成功了，不再出现停五分钟不用无响应的问题。至于为什么一开始不用FuckDoctors的EFI，因为一加载就ACPI错误黑屏，可能他没用热补丁或者我机器DSDT和他的差别大，然后就一直没用。至于如何导出编译本机DSDT可以参考这里：http://bbs.pcbeta.com/viewthread-1571455-1-1.html  。

具体操作中遇到的问题：我把我反编译出的DSDT.dsl用工具MaciASL打开点编译，发现里面只有一种错误：Firmware Error (ACPI): Failure looking up，我把它们全删掉，重新编译成功，然后就可以对DSDT做修改了。我上传的DSDT里对针对触摸板修改的地方都已加上了注释：patches for VoodooI2C.kext 有需要修改自己DSDT的可以自己对比查看。 如果谁用我的DSDT进不去系统，可以先把我patch下的DSDT删除，不用我的DSDT只用已有的热补丁的话应该百分之九十能进去系统，如果发现触摸板有同样问题，可以再提取自己本机的DSDT做修改解决，总之，用别人的DSDT很可能进不去系统。



