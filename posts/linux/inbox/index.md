# 记录一些日常遇到的问题和解决方法



问题： `tail: inotify cannot be used, reverting to polling: Too many open files` ?

解决：在 Linux 系统中，`tail`可以使用`inotify`机制来高效地监控文件的变化。`inotify`不能被使用的时候，会退回到轮询（polling）模式来检测文件变化。
当打开文件数超过系统限制时，就无法在分配更多的资源给`inotify`来跟踪文件的变化。

1. 找出打开过多文件的进程, `lsof` 命令:
```shell
[root@localhost ~]# lsof &#43;c 0 | awk &#39;{print $2}&#39; | sort | uniq -c | sort -nr | head
  16832 8921
  12481 3046
   5880 10266
# 输出结果中，第一列是进程号，第二列是进程名。
```
如果是自己的程序，检查代码逻辑； 如果是系统或第三方进程，尝试重启来释放占用的资源（谨慎！！！）；
2. 调整系统文件描述符限制， `ulimit` 命令查看:
```shell
[root@localhost ~]# ulimit -n  # 查看软限制
65535 
[root@localhost ~]# ulimit -Hn   # 查看硬限制
65535
```
- 硬限制是系统管理员设置的上限，软限制不能超过硬限制;
- 在当前会话终端中，执行 `ulimint -n [NUMBER]` 命令，修改这个限制，但只对当前会话有效；
- 如果需要永久调整限制，通过编辑 `/etc/security/limits.conf`文件，修改完成后重新登录系统就会生效；



---

> 作者: 迷途小狼崽  
> URL: http://localhost:1313/posts/linux/inbox/  

