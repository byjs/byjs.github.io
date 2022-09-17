---
title: JavaScript - requestAnimationFrame

date: 2019-11-21

tag: JavaScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/a8e180bb17a9f555c05c0bf9045b5fe25f921faa.jpg
---

# 关于requestAnimationFrame

| 数据   | 渲染间隔   |
|------|--------|
| 10ms | 16.7ms |
| 20ms |        |
| 30ms | 33.4ms |
| 40ms |        |
| 50ms | 50.1ms |
| 60ms | 66.8ms |
| 70ms |        |
| 80ms | 83.5ms |

- 假设场景：国庆高速，最多每16.7s通过一辆车，突然插入一批setTimeout的车，强行10s通过。就超负荷了
- 显示器 16.7ms 刷新间隔之间发生了其他绘制请求，导致部分帧消失，导致动画断续显示，这是过度绘制带来的问题，所以 setTimeout的定时器值推荐最小使用16.7ms（16.7 = 1000/60 每秒60帧）
- requestAnimationFrame：跟着浏览器的绘制走（浏览器绘制间隔10ms，就10ms绘制），这样就不会有过度绘制的问题，动画不会掉帧


- 总结setTimeout 与 requestAnimationFrame 的区别：
    - 引擎层面：setTimeout 属于 JS 引擎，存在事件轮询，存在事件队列。requestAnimationFrame 属于 GUI 引擎，发生在渲染过程的中重绘重排部分，与电脑分辨路保持一致
    - 性能层面：当页面被隐藏或最小化时，定时器 setTimeout 仍在后台执行动画任务。当页面处于未激活的状态下，该页面的屏幕刷新任务会被系统暂停，requestAnimationFrame 也会停止
    - 应用层面：利用 setTimeout，这种定时机制去做动画，模拟固定时间刷新页面。requestAnimationFrame 由浏览器专门为动画提供的
      API，在运行时浏览器会自动优化方法的调用，在特定性环境下可以有效节省了CPU 开销

## 深究

- 宏任务：主js、UI渲染、setTimeout、setInterval、setImmediate、requestAnimationFarme、I/O
- 微任务：process.nextTick、promise、Object.observe等