- ## 信号的使用
- ##### stop vs quit 
  stop发送 SIGTERM 信号，表示要求强制退出，quit 发送 SIGQUIT，表示优雅地退出。 具体区别在于，worker 进程在收到 SIGQUIT 消息(注意不是直接发送信号，所以这里用消息替代)后，会关闭监听的套接字，关闭当前空闲的连接(可以被抢占的连接)，然后提前处理所有的定时器事件，最后退出。没有特殊情况，都应该使用 quit 而不是 stop。
-
- ##### reload
  master 进程收到 SIGHUP 后，会重新进行配置文件解析、共享内存申请，等一系列其他的工作，然后产生一批新的 worker 进程，最后向旧的 worker 进程发送 SIGQUIT 对应的消息，最终无缝实现了重启操作。 再 master 进程重新解析配置文件过程中，如果解析失败则会回滚使用原来的配置文件，即 reload 失败，此时工作的还是老的 worker。
- reopen
  master 进程收到 SIGUSR1 后，会重新打开所有已经打开的文件(比如日志)，然后向每个 worker 进程发送 SIGUSR1 信息，worker 进程收到信号后，会执行同样的操作。reopen 可用于日志切割，比如 nginx 官方就提供了一个方案：
  ```shell
   $ mv access.log access.log.0
   $ kill -USR1 `cat master.nginx.pid`
   $ sleep 1
   $ gzip access.log.0    # do something with access.log.0
  ```
- 这里 sleep 1 是必须的，因为在 master 进程向 worker 进程发送 SIGUSR1 消息到 worker 进程真正重新打开 access.log 之间，有一段时间窗口，此时 worker 进程还是向文件 access.log.0 里写入日志的。通过 sleep 1s，保证了 access.log.0 日志信息的完整性(如果没有 sleep 而直接进行压缩，很有可能出现日志丢失的情况)。
- ##### hot update
  某些时候我们需要进行二进制热更新，nginx 在设计的时候就包含了这种功能，不过无法通过 nginx 提供的命令行完成，我们需要手动发送信号。首先需要给当前的 master 进程发送 SIGUSR2，之后 master 会重命名 nginx.pid 到 nginx.pid.oldbin，然后 fork 一个新的进程，新进程会通过 execve 这个系统调用，使用新的 nginx ELF 文件替换当前的进程映像，成为新的 master 进程。新 master 进程起来之后，就会进行配置文件解析等操作，然后 fork 出新的 worker 进程开始工作。接着我们向旧的 master 发送 SIGWINCH 信号，然后旧的 master 进程则会向它的 worker 进程发送 SIGQUIT 信息，从而使得 worker 进程退出。向 master 进程发送 SIGWINCH 和 SIGQUIT 都会使得 worker 进程退出，但是前者不会使得 master 进程也退出。最后，如果我们觉得旧的 master 进程使命完成，就可以向它发送 SIGQUIT 信号，让其退出了