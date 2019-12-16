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

       开启宽容模式，修改/etc/selinux/config中的SELINUX=permissive，然后重启。

       开启强制模式，修改/etc/selinux/config中的SELINUX=enforcing，然后重启。
       *注意* 切换到强制模式后，selinux可能会拒绝一些操作，因为错误的或者缺失的规则。
       可以通过ausearch -m AVC,USER_AVC,SELINUX_ERR -ts today查看拒绝的操作。
       如果安装了setroubleshoot-server，可以输入grep "SELinux is preventing" /var/log/messages。

       不开启selinux，修改/etc/selinux/config中的SELINUX=disabled，然后重启系统。

  * 在启动时改变SELINUX模式

    可以添加下列内核参数，改变SELinux运行模式：

      1 enforcing=0设置这个参数导致机器进入宽容模式，这在排查故障时非常有用。强制模式拒绝访问的操作，
      在宽容模式下，会产生AVC信息，只有第一次拒绝会被通知。

      2 selinux=0设置这个参数导致内核不加载selinux任何功能。init脚本通知内核用selinux=0启动，然后touch /.autorelabel文件。
      /.autorelabel文件在下一次开启selinux系统启动时自动重启给文件系统打标签。不推荐用selinux=0，debug系统，建议用宽容模式。

      3 autorelabel=1，和touch /.autorelabel; reboot功能相同。

      还有一些其他的参数，查看内核文档进行使用。

  * 开关

    开关允许在运行时改变selinux策略的部分功能，不需要了解任何selinux的编写策略。这允许改变，比如在没有重新加载或者重新编译selinux
    策略的情况下，允许服务访问nfs文件系统。

    1 在root下，semanage boolean -l可以列出所有开关。getsebool -a可以看到所有开关当前状态。

    2 通过setsebool [boolean_name] on/off可以进行开关的配置。

    3 通过setsebool -P [boolean_name] on/off可以使开关在重启后持久。

    4 shell自动补齐，getsebool、setsebool、semanage可以使用自动补齐。

  * selinux上下文——标签文件

    selinux规则在DAC（自由访问控制）规则之后，如果DAC拒绝访问，那么selinux规则不会被用到。
    *注意* 默认情况下，新生成的文件和目录会继承父目录的selinux类型。有多个命令可以管理selinux上下文，比如chcon、semanage、fcontext、restorecon。

    1 临时更改:注意chcon修改后，在autorelabel或者restorecon之后，就失效了。
    快速参考：chcon -t httpd_sys_content_t [file-name]；chcon -t -R httpd_sys_content_t [dir-name]，
    可以使用restorecon [file]命令恢复默认上下文。默认上下文配置文件存放在/etc/selinux/targeted/contexts/files/目录下。

    2 持久更改：semanage fcontext -C -l；当文件系统relabeled的时候，setfiles工具被用。重新存储默认上下文用restorecon命令。
    快速参考：semanage fcontext -a options [file-name|directory-name]；restorecon -v [file-name|directory-name]
    *主要* semanage fcontext -a 需要使用全路径，不然可能错误的标记文件。

- sepolicy组件

- 限制用户

- 用沙盒的安全程序

- svirt

- 安全linux容器

- selinux systemd访问控制

- 故障排除

- 深入了解