###Tips

####解决SecureCRT连接linux超时后断开

	* 客户端：Options -> Session Options -> Terminal, 勾选“Send protocol NO-OP”后, 设置超时时间
	* 服务端：修改/etc/ssh/sshd_config配置文件 ClientAliveInterval 300（默认为0，单位为秒，然后用service sshd reload生效

###终端分屏工具tmux

	http://blog.jobbole.com/87584/

###zip和unzip

	* zip: zip -r my.zip my/
	* unzip: unzip my.zip -d my/

###Nginx打开目录浏览功能

	location / {
		root  /home/yinjiongjie/work;
		autoindex on;
		#以下参数可选
		autoindex_localtime on; #显示文件时间
		autoindex_exact_size off; #显示文件大小
	}

###NERDTree快捷键

####目录切换
* ctrl + w + h 光标 focus 左侧树形目录
* ctrl + w + l 光标 focus 右侧文件显示窗口
* ctrl + w + w 光标自动在左右侧窗口切换
* ctrl + w + r 移动当前窗口的布局位置

### CPU核数
* grep processor /proc/cpuinfo | wc -l

### 删除本地过期分支

	git remote prune origin

### 更新远程分支

	git fetch
