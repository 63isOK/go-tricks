# 遍历多通道

```go
func (a *agollo) sendWatchCh(namespace string, oldVal, newVal Configurations) {
	changes := oldVal.Different(newVal)
	if len(changes) == 0 {
		return
	}

	resp := &ApolloResponse{
		Namespace: namespace,
		OldValue:  oldVal,
		NewValue:  newVal,
		Changes:   changes,
	}

	timer := time.NewTimer(defaultWatchTimeout)
	for _, watchCh := range a.getWatchChs(namespace) {
		select {
		case watchCh <- resp:

		case <-timer.C: // 防止创建全局监听或者某个namespace监听却不消费死锁问题
			timer.Reset(defaultWatchTimeout)
		}
	}
}
```

来自[agollo项目](https://github.com/shima-park/agollo)

## 解析

select会阻塞,用定时器就可以可推出循环,用for循环遍历多通道,
这样就可以实现遍历多通道,遍历间隔就是定时器的值.
