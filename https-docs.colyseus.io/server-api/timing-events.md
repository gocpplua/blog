# Timing events

## 计时事件

对于[计时事件](https://www.w3.org/TR/2011/WD-html5-20110525/timers.html)，建议使用Room实例中的[`this.clock`](https://docs.colyseus.io/server/room/#clock-clocktimer)方法。

清理房间后，“ this.clock”上注册的所有间隔和超时都将自动清除。

内置的setTimeout和setInterval方法依赖于CPU负载，这可能会延迟意外的执行时间。

### Clock <a id="clock"></a>

时钟是一种有用的机制，用于在有状态模拟之外对事件计时。

#### 公共方法 <a id="public-methods"></a>

_注意：`time`参数以毫秒为单位_

**clock.setInterval\(callback, time, ...args\): Delayed**

`setInterval()`方法重复调用函数或执行代码段，每次调用之间有固定的时间延迟。它返回 [`Delayed`](https://docs.colyseus.io/server/timing-events/#delayed)标识间隔的实例，因此您以后可以对其进行操作。

**clock.setTimeout\(callback, time, ...args\): Delayed**

`setTimeout()`方法设置一个计时器，该计时器在计时器到期后执行一次功能或指定的代码段。它返回[`Delayed`](https://docs.colyseus.io/server/timing-events/#delayed) 标识间隔的实例，因此您以后可以对其进行操作。

此MVP示例显示了一个Room ：`setInterval()`，`setTimeout`并清除了先前存储的type实例`Delayed`；以及显示Room时钟实例中的currentTime。1秒“现在时间” +后`this.clock.currentTime`是`console.log`“d，然后在10秒后，我们清除间隔：`this.delayedInterval.clear();`。

```text
// Import Delayed
import { Room, Client, Delayed } from "colyseus";

export class MyRoom extends Room {
    // For this example
    public delayedInterval!: Delayed;

    // When room is initialized
    onCreate(options: any) {
        // start the clock ticking
        this.clock.start();

        // Set an interval and store a reference to it
        // so that we may clear it later
        this.delayedInterval = this.clock.setInterval(() => {
            console.log("Time now " + this.clock.currentTime);
        }, 1000);

        // After 10 seconds clear the timeout;
        // this will *stop and destroy* the timeout completely
        this.clock.setTimeout(() => {
            this.delayedInterval.clear();
        }, 10_000);
    }
}
```

**clock.clear\(\)**

{% embed url="https://清除clock.setInterval（）和clock.setTimeout（）注册的所有间隔和超时。" %}

**clock.start\(\)**

开始计时。

**clock.stop\(\)**

停止计时。

**clock.tick\(\)**

在每个模拟间隔步骤都会自动调用此方法。 在Tick中检查所有延迟的实例。

> 有关更多详细信息，请参见[Room＃setSimiulationInterval（）](https://docs.colyseus.io/server/room/#setsimulationinterval-callback-milliseconds166)。

#### 公共属性 <a id="public-properties"></a>

**clock.elapsedTime**

自从[`clock.start()`](https://docs.colyseus.io/server/timing-events/#clockstart)调用方法以来经过的时间（以毫秒为单位）。只读。

**clock.currentTime**

当前时间（以毫秒为单位）。只读。

**clock.deltaTime**

上次`clock.tick()`通话与当前通话之间的毫秒数差。只读。

### 延迟Delayed <a id="delayed"></a>

延迟的实例是通过[`clock.setInterval()`](https://docs.colyseus.io/server/timing-events/#clocksetintervalcallback-time-args-delayed)或 [`clock.setTimeout()`](https://docs.colyseus.io/server/timing-events/#clocksettimeoutcallback-time-args-delayed)方法创建的 。

#### 公开方法[¶](https://docs.colyseus.io/server/timing-events/#public-methods_1) <a id="public-methods_1"></a>

**delayed.pause\(\)**

暂停特定`Delayed`实例的时间。（`elapsedTime`直到`.resume()`被调用之前，它不会增加。）

**delayed.resume\(\)**

恢复特定`Delayed`实例的时间。（`elapsedTime`通常会继续增加）

**delayed.clear\(\)**

清除超时或间隔。

**delayed.reset\(\)**

重置经过的时间。

#### 公共属性 <a id="public-properties_1"></a>

**delayed.elapsedTime: number**

`Delayed`实例经过的时间（自启动以来的毫秒数）。

**delayed.active: boolean**

返回`true`计时器是否仍在运行。

**delayed.paused: boolean**

返回`true`计时器是否已通过暂停`.pause()`。



