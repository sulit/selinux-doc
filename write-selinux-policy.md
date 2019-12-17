编写SELinux策略
=============

- selinux模式

  enforcing
  selinux安全策略被内核强制执行。

  permissive
  selinux安全策略不被内核强制执行。访问是被日志记录的。

- 切换selinux模式到premissive以便收集更多的AVC信息

  AVC信息记录在/var/log/audit/audit.log中。
  ausearch -m AVC可以查看关于AVC的信息。
  可以通过sesearch和audit2allow命令解析AVC信息。
  ausearch -m AVC [-ts today|recent] | audit2allow

- 编写通用策略需要的所有的必要的文件可以在/USR/SHARE/SELINUX/DEVEL目录下发现。

- 一个关于rhsummit.service的例子

  * 用于测试的虚拟linux守护进程，我们将为它编写策略，干！！！

  * 这个守护进程访问一个网址的80端口；写日志进journal；生成pid文件；读/proc/meminfo。

  * 没有为rhsummit服务的selinux策略意味着这个服务是未受限的服务。
    #ps -efZ | grep rhsummit
    system_u:system_r:unconfined_service_t:s0 root 2828 1 0 16:16 ? 00:00:00 /usr/bin/rhsummit

  * #sepolicy generate --init /usr/bin/rhsummit
    Loaded plugins: product-id
    Created the following files:
    /root/code/policy/rhsummit.te # Type Enforcement file
    /root/code/policy/rhsummit.if # Interface file
    /root/code/policy/rhsummit.fc # File Contexts file
    /root/code/policy/rhsummit_selinux.spec # Spec file
    /root/code/policy/rhsummit.sh # Setup Script

  * #./rhsummit.sh

  * 如何检查我们summit服务的selinux状态
    # systemctl stop rhsummit
    # systemctl start rhsummit
    # ps -efZ | grep summit
    system_u:system_r:rhsummit_t:s0 root 3049 1 0 17:03 ? 00:00:00 /usr/bin/rhsummit

  * rhsummit服务以rhsummit标签运行，意味着这个服务是受限制的。

  * 重要的selinux策略文件

    1 rhsummit.fc 文件包含了所有服务目标文件的selinux安全上下文

    2 rhsummit.te 文件包含了rhsummit_t域的规则

    3 rhsummit.if 文件包含了访问rhsummit_t类型的接口

    4 rhsummit.sh 是rhsummit服务编译和安装selinux策略的脚本
  