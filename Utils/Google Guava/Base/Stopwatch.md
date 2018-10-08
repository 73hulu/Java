# Stopwatch
计时器。`System.currentTime()`的封装版本。用在代码里提高level。


![Stopwatch](https://ws2.sinaimg.cn/large/006tNbRwly1fw0t6agc3bj30iw0miq3a.jpg)

使用示例：

```Java
Stopwatch stopwatch = Stopwatch.createStarted();
doSomething();
stopwatch.stop(); // optional

Duration duration = stopwatch.elapsed();

log.info("time: " + stopwatch); // formatted string like "12.3 ms"
```
