title:: Nagle算法/SO_NODELAY

- [linux官方文档](https://linux.die.net/man/7/tcp)
- 但记住一点，Nagle算法是时代的产物，因为当时网络带宽有限。而当前的局域网、广域网的带宽则宽裕得多，所以目前的TCP/IP协议栈默认将Nagle算法关闭，即通过SO_NODELAY = 1
-
- > TCP_NODELAY
  If set, disable the Nagle algorithm. This means that segments are always sent as soon as possible, even if there is only a small amount of data. When not set, data is buffered until there is a sufficient amount to send out, thereby avoiding the frequent sending of small packets, which results in poor utilization of the network. This option is overridden by TCP_CORK; however, setting this option forces an explicit flush of pending output, even if TCP_CORK is currently set.