+++
date = "2015-11-27T15:24:54+08:00"
draft = false 
title = "Ubuntu调节屏幕亮度"
tags = ["ubuntu"]

+++

# Ubuntu14.10调节屏幕亮度

命令行输入`sudo vim /etc/default/grub`<br> 
找到<br>
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"  
GRUB_CMDLINE_LINUX=""

改为<br>
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_osi=Linux"<br>
GRUB_CMDLINE_LINUX="acpi_osi=Linux acpi_backlight=vendor"  

保存

升级grub `sudo update-grub`<br>
之后重启电脑就可以用Fn调节屏幕亮度了。

###  补充
设定亮度默认值

终端输入 `sudo gedit /etc/rc.local` 
在打开文件里增加一句（加在exit 0之前） 
代码: `echo 500 > /sys/class/backlight/intel_backlight/brightness`  
意思是默认值设为500，可更改为其他合适的数字。 
最后保存
