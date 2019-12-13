selinux用户和管理员手册
====================

从Red_Hat_Enterprise_Linux-7-SELinux_Users_and_Administrators_Guide-en-US（Last Updated: 2019-09-02）整理

- 介绍

- selinux上下文

- 针对性策略(targeted policy)

- 使用selinux

  * 这章主要介绍，在红帽企业版linux上主要的selinux包、安装和更新软件包、哪些日志文件被用、主要的selinux配置文件、开启和关闭selinux功能、selinux的模式、配置开关、临时和持久地改变文件和目录标签、用mount命令覆盖文件系统标签、挂载网络文件系统卷、当复制和打包文件和目录时如何保留selinux上下文。
  * selinux相关的软件包

     默认selinux运行在enforcing模式，针对性（targeted）策略被用。

     policycoreutils的用于管理selinux的工具restorecon、secon、setfiles、semodule、load_policy、setsebool。

     selinux-policy提供基本的目录结构、selinux-policy.conf和一些rpm宏。

     selinux-policy-targeted提供selinux针对性策略。

     libselinux提供开发selinux应用的API

     libselinux-utils提供的工具avcstat、getenforce、getsebool、matchpathcon、selinuxconlist、selinuxdelcon、selinuxenabled、setenforce。

     selinux-policy-mls提供MLS（Multi-Level）selinux策略。

     setools-console提供了大量的工具和库用于分析查询策略、日志监控和报告和文件上下文管理等。包含sechecker、sediff、seinfo、sesearch、findcon等。

     policycoreutils-python提供了semanage、audit2allow、audit2why、chcat。

  * 哪些日志文件被用。

     需要安装setroubleshoot-server。（感觉不是必须的）

     /var/log/audit/audit.log /var/log/message

  * 主要配置文件

     /etc/selinux/config

     SELINUX=(enforcing、permissive、diabled)模式类型

     SELINUXTYPE=(targeted作为默认策略、mls等)

  * 永久性的改变selinux状态和模式

    *注意* 当系统selinux运行在宽松模式下时，用户可以不正确的给文件打标签。当selinux不开启时，生成文件是不会被打标签的。
    当selinux改变到强制模式时，这两种行为都会导致问题。为了避免这些行为，当selinux状态和模式改变时，文件系统会自动重新打标签。

    1 开启selinux

       推荐下列流程：开启permissive模式，然后重启系统，检查selinux否认信息，如果没有否认信息，切换到强制模式。

       在强制模式下运行自己开发的应用：首先在unconfined_service_t下运行，然后为自己的应用写一个新的策略。

- sepolicy组件

- 限制用户

- 用沙盒的安全程序

- svirt

- 安全linux容器

- selinux systemd访问控制

- 故障排除

- 深入了解