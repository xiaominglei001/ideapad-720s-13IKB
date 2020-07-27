##  ideapad-720s-13IKB
我买的日版机，CPU是i5 8250u，其它参数可看鲁大师导出的配置单，大部分文件来自作者dragonflylee的仓库   https://github.com/dragonflylee/Yoga-730-hackintosh

###  重点使用方法：
1. 用黑果小兵提供的10.15.3镜像，刻录一个安装盘，用此处的EFI替换安装U盘里的EFI，然后可以用此盘来安装系统。
2. bios设置参考原作者的，Disable Security -> Intel SGX -> Intel SGX Controller   Disable Security -> Secure Boot  Switch RAID to AHCI in Configuration -> SATA Controller Mode，720s有的选项就照他的改。注意进入bios方法，冷启动后快速连按F2。
3. 系统安装时最好把硬盘格式化为APFS格式GUID分区，找不到硬盘的在硬盘工具左上方切换为“显示所有设备”，安装完成后，再用此处的EFI替换系统盘里的EFI，关机拔掉U盘，开机设置启动顺序从系统盘启动，即可脱离u盘启动。注意开机顺序的问题，如果设置错了可能出现enumerating for asix usb dongle...等，这时可以重置bios之后再重新设置。
4. 发现问题根据需要再改config.plist，如声卡layoutid。可能值为 3, 11, 13, 21, 22, 28, 29, 30, 32, 47, 66, 72, 99。




### 关于折腾的一些记录
#### 关于触摸板bug修复
######  发现触摸板问题
用原作者的EFI发现触摸板在闲置一段时间，如5分钟多点，就会不响应，像完全没驱动一样，必须重启才可以正常，可能需要自己定制DSDT(或许voodooi2c本身有bug)参考教程：https://www.penghubingzhou.cn/2019/01/06/VoodooI2C%20DSDT%20Edit/ 或官方说明  https://voodooi2c.github.io。

######  分析问题
仔细读了官方文档 https://voodooi2c.github.io 发现官方文档写的还是很清晰的，终于有点明白了，总的来说就是先对DSDT打补丁再根据自己触摸设备需要调整GPIO Pinning，然后我看了下dragonflylee原来的EFI，发现作者没有修改DSDT貌似，只是热补丁修改SSDT，当然这两者具体关系我也不太懂。所以现在的问题是我要按voodooi2c官方文档要求修复一份DSDT放进去，然后我又想到这个作者FuckDoctors（ 他EFI地址 https://github.com/FuckDoctors/ideapad-720s-13IKB ） 在issue里说他的DSDT只修改了触摸板和调节亮度的部分，就想可以参照着他的改。

###### 修复步骤
大致步骤：我把FuckDoctors的DSDT.dsl下载下来搜触摸板bios设备名：TPD0，然后找到他所有为voodooi2c打补丁的地方（作者还用zhbchwin注释了这里地方）以待使用，然后我在Clover界面按F4提取我自己的文件，然后用iasl反编译，反编译时只把DSDT和SSDT开头的文件放到同一个文件夹里去反编译,否则反编译时报错，因为其它文件反编译不了，之后再用maciasl修改反编译后的DSDT.dsl，参照FuckDoctors的DSDT.dsl把需要的代码挪到我自己的DSDT.dsl里，然后编译导出DSDT.aml，放到我patch里，然后就成功了，不再出现停五分钟不用无响应的问题。至于为什么一开始不用FuckDoctors的EFI，因为一加载就ACPI错误黑屏，可能他没用热补丁或者我机器DSDT和他的差别大，然后就一直没用。至于更详细的导出编译本机DSDT方法可以参考这里：http://bbs.pcbeta.com/viewthread-1571455-1-1.html  。

具体操作中遇到的问题：我把我反编译出的DSDT.dsl用工具MaciASL打开点编译，发现里面只有一种错误：Firmware Error (ACPI): Failure looking up....，我参照FuckDoctors的DSDT.dsl把它们全删掉，重新编译成功，然后就可以对DSDT.dsl做修改了。我上传的DSDT.aml里对针对触摸板修改的地方都已加上了注释：patches for VoodooI2C.kext 有需要修改自己DSDT的可以自己对比查看。 如果谁用我的DSDT进不去系统，可以先把我patch下的DSDT删除，不用我的DSDT只用已有的热补丁的话应该百分之九十能进去系统，如果发现触摸板有同样问题，可以再提取自己本机的DSDT做修改解决，总之，用别人的DSDT很可能进不去系统。


#### 关于网卡
######  前期：usb网卡及dw1820a、bcm94360cs2
开始用的USB迷你网卡comfast cf-811ac，驱动双击安装即可（驱动更新地址github搜：Wireless-USB-Adapter-Clover）和安卓手机分享网络（需要HoRNDIS_USB.kext驱动），comfast cf-811ac和安卓USB分享网络好像同时只能用一个，要换另一个的话可能要重新设置下再重启电脑。但鉴于占用宝贵的USB口，我最终还是买了dw1820a，但是系统就是识别不出来，无论win下还是mac下，可能是网卡白名单问题，无论屏蔽针脚与否或者先bios关网卡再打开都不行，就又买了bcm94360cs2加仕和技术的m2转苹果网卡那个延长线，把电池拿下来去掉扬声器，可以很好的放入这个网卡，反正扬声器几乎不用，用这个网卡时记得先bios里屏蔽wifi进系统，然后再解开屏蔽进系统，bcm94360cs2直接免驱，可以把相关驱动删了(AirportBrcmFixup.kext、BT4LEContinuityFixup.kext、BrcmBluetoothInjector.kext)。


######  又更新为BCM94352Z联想版
原来的转接线居然烧了。。。冒烟。。所以，感觉这个方案也不靠谱，主要空间太小了，线被压折的狠了，绝缘出现问题，所以又入手了个BCM94352Z联想版。收到后，又新下了一遍https://github.com/dragonflylee/Yoga-730-hackintosh 中的文件，因为原作者也更新了，然后先bios屏蔽网卡进入系统，再进入系统EFI放入下到的文件后，再关机进入bios打开网卡进入系统，此时网卡蓝牙应该都驱动了。如果不能登录appstore需要网卡内建，删除/Library/Preferences/SystemConfiguration/NetworkInterfaces.plist然后在系统设置偏好下网络里删除所有网络连接，点"应用"然后重启。然后other驱动目录放入HoRNDIS_USB.kext，方便用安卓手机连接电脑共享网络，放入comfast cf-811ac网卡安装时的驱动方便以后万一要用外置网卡，还把以前修改的patch里的文件拷入EFI相应位置解决触摸板问题（拷入前先删掉patch里面原来文件），最后还是放入自己之前用的config.plist，因为直接用原作者的，会出现一些问题，如亮度调节及触摸板驱动等，具体这个config.plist的设置我也忘了，记得改了声卡layoutid为22，及修复关机问题，具体功能查百度吧，目前我用的这个EFI各项都正常：网卡，亮度，显卡，关机，输入等等，想要真正百分百完美肯定有距离的，就这样先用着吧，暂时不折腾了，达到需求了。


### 其它说明
最新的EFI已打包上传了，有问题需要讨论可去 http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1817262&highlight=720s  我只是看到讨论里用这个EFI可以，就参考用了,其它很多也不懂，比如config.plist的设置等，有问题多百度吧。



我的文件地址(供外网参考)：https://github.com/xiaominglei001/ideapad-720s-13IKB
