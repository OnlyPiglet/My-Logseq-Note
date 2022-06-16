- [[redis协议]]
- [[redis主从集群]]
- [[redis 哨兵]]
- [[redis书籍]]
	- ((6235af2c-6f5c-4d35-8c63-05d5d81be88f))
	- ((6235af51-9d9d-414b-b012-359b38acac4b))
- ## redis lua代码实现 分布式锁
- ```python
  def acquire_lock(conn,lockname,timeout=10,lock_timeout):
    identifier = str(uuid.uuid4())
    end_time = time.time() + timeout
    locl_timeout = int(math.ceil(lock_timeout))
    while time.time() < end:
      ## 首次进入需要设置过期时间进行释放
      if not conn.ttl(lockname):
        conn.expire(lockname,locl_timeout)
      if conn.setnx('lock:'+lockname,identifier):
        return identifier
      time.sleep(.001)
    return False
  
  
  def release_lock(conn,lockname,identifier):
    pipe = conn.pipeline(True)
    lockname = 'lock:'+lockname
    while Ture:
      try:
        pipe.watch(lockname)  #watch 是一个乐观锁 当lockname被其链接所修改的时候 会抛出异常
        if pipe.get(lockname) == identifier:  # 当lockname 取不到 identifier 或者 不等于自己设置的identifier时表示 已经过期 此时应该直接 unwatch 并结束退出 避免死循环
          pipe.multi()
          pipe.delete(locknanme)
          pipe.execute()
          return True
        pipe.unwatch()
        break
      except: redis.exceptions.WatchError:         #发生异常表示有其他客户端修改了lockname 
        pass  #重新watch并进行相关处理
    return False
  ```