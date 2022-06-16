- [Go语言的time包](https://zhuanlan.zhihu.com/p/357348340)
- ## 10位 字符串数组 与 time.Time转换
- ```go
  func SecondToTime(e string) (datatime time.Time, err error) {
  	data, err := strconv.ParseInt(e, 10, 64)
  	datatime = time.Unix(data, 0)
  	return
  }
  
  func TimeToSecond(e time.Time) int64 {
  	timeUnix, _ := time.Parse("2006-01-02 15:04:05", e.Format("2006-01-02 15:04:05"))
  	return timeUnix.UnixNano() / 1e9
  }
  ```