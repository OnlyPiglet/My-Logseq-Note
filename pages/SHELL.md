- ## 显示不同底色脚本
- ```shell
  #!/bin/sh
  
  ver=$1
  
  red='\e[0;41m'
  RED='\e[1;31m'
  green='\e[0;32m'
  GREEN='\e[1;32m'
  NC='\e[0m'
  
  echo -e "${red}not passed"
  echo -e "${RED}not passed"
  echo -e "${green}passed"
  echo -e "${GREEN}passed"
  echo -e "${NC}reset"
  ```
- ## 显示不同操作系统类型
- ```shell
  #!/bin/sh
  
  if grep -q -i 'Fedora' /etc/redhat-release; then
      echo "fedora"
  
  elif grep -q -i 'CentOS' /etc/redhat-release; then
      echo "centos"
  
  elif grep -q -i 'Red Hat' /etc/redhat-release; then
      echo "rhel"
  
  elif grep -q -i 'Alibaba Cloud Linux' /etc/redhat-release; then
      echo "alinux"
  
  else
      echo "unsupported redhat-based os." > /dev/stderr
      _fail
  fi
  ```
- {{embed [[源码打包命令]]}}
- # su 与 sudo
	- su (switch user)
		- **su - <user_name>**   不保留当前环境变量设置 切换至新用户
		-
		- ```shell
		  PWD=/home/ubuntu  # 是 /home/ubuntu  
		  HOME=/home/ubuntu    
		  
		  ubuntu@VM-0-14-ubuntu:~$ su - root   # 是 login-shell 方式  
		  
		  PWD=/root   # 已经变成 /root 了  
		  HOME=/root  
		  ```
		- **su <user_name>** 保留当前环境变量设置 切换至新用户
		- ```shell
		  PWD=/home/ubuntu    # 是 /home/ubuntu  
		  HOME=/home/ubuntu  
		  
		  ubuntu@VM-0-14-ubuntu:~$ su root   # non-login-shell 方式  
		  
		  PWD=/home/ubuntu  # 可以发现还是 /home/ubuntu  
		  HOME=/home/ubuntu  
		  
		  ```
	- sudo (super user do)
		- > 一个用户能否使用 sudo 命令，取决于 /etc/sudoers 文件的设置，/etc/sudoers 也是一个文本文件，但是因其有特定的语法，我们不要直接用 vim 或者 vi 来编辑它，需要用 visudo 这个命令。输入这个命令之后就能直接编辑 /etc/sudoers 这个文件了。需要说明的是，只有 root 用户有权限使用 visudo 命令。我们先来看下输入 visudo 命令后显示的内容。
		- ```
		  # User privilege specification  
		  root    ALL=(ALL:ALL) ALL  
		    
		  # Members of the admin group may gain root privileges  
		  %admin ALL=(ALL) ALL  
		    
		  # Allow members of group sudo to execute any command  
		  %sudo   ALL=(ALL:ALL) ALL  
		    
		  # See sudoers(5) for more information on "#include" directives:  
		    
		  #includedir /etc/sudoers.d  
		  ubuntu  ALL=(ALL:ALL) NOPASSWD: ALL  
		  ```
	- 常见工具
		- ![image.png](../assets/image_1652962521580_0.png)
		-