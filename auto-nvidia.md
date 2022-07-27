# 自动检测nvidia驱动是否掉卡

```bash
if type nvidia-smi >/dev/null 2>&1; then
	echo -e "\033[32m The video card is running...  \033[0m"

else
	echo -e "\033[33m Driver not installed  \033[0m" >/logs.log

	echo -e "\033[33m Clear the running nVidia kernel\033[0m"
	sudo fuser -v /dev/nvidia* | awk '{for(i=1;i<=NF;i++)print "kill -9 " $i;}' | sudo sh

	echo -e "\033[33m The driver installation starts\033[0m"
	/home/smore/NVIDIA-Linux-x86_64-510.60.02.run -Z -z -a -s --no-opengl-files >>/logs.log
	echo -e "\033[32m Successful installation... \033[0m"
fi
```


# 添加定时任务

`crontab -e`

# 每两小时执行一次

` * 2 * *  * /bin/bash "/home/smore/check.sh" >> /logs.log`