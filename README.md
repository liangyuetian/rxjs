### 首先，函数式编程
顾名思义,函数式编程就是非常强调使用函数来解决问题的一种编程方式。
要求以下几点：
* 声明式
* 纯函数
* 数据不可变性

### Observable 和 Observer 
* observable: 可以被观察的对象，即可被观察者
* observer: 观察者

### RxJS中的数据流就是Observable对象，Observable实现了下面两种设计模式：
* 观察者模式
* 迭代器模式

### Observable 的核心在于各种各样的操作符
操作符分为以下几大类：
* 创建操作符
* 合并操作符
* 辅助类操作符
* 过滤操作符
* 转化操作符
* 异常错误处理操作符

### 还有
* 多播
* 调度器(Scheduler)
* 高阶Observable
* rxjs的调试和测试

## 第一大类：创建操作符

| 操作符 | 功能 |
| ------ | ------ |
| create | 直接创建观察者对象 |
| of | 根据参数直接产生同步数据 |
| range | 产生一个数值范围内的数据 |
| generate | 以循环方式产生数据，相当于for循环，有判断条件 |
| repeat repeatWhen | 重复产生数据流中的数据 |
| empty | 产生空的数据流，并结束 |
| throw | 产生直接出错的数据流，并结束 |
| never | 产生永不完结的数据流 |
| interval timer | 间隔给定时间持续产生数据流 |
| from | 从数组等枚举类型数据产生数据流 |
| fromPromise | 从Promise中产生数据流 |
| fromEvent fromEventPattern | 从外部事件对象产生数据流 |
| ajax | 从ajax请求产生数据流 |
| defer | 延迟产生数据流 |

例子：
#### create 毫无特点
```js
Observable.create = function(subscribe) {
    return new Observable(subscribe)
}
```

#### of 列举数据
```js
const source$ = Observable.of(1, 2, 3);
source$.subscribe(
    console.log, // 收到数据时触发的回调
    null, // 相当于catch
    () => console.log('complete') // 当整个数据流结束时触发
) // result: 1 2 3 同步产生
```

#### range 指定范围
```js
const source$ = Observable.range(1, 100);
// 订阅后直接输出1 2 3 4 5 ... 99 100 同步产生
```

#### generate 循环创建
```js
const source$ = Observable.generate(
    2, // 初始值
    value => value < 10, // 继续的条件，相当于for中的条件判断
    value => value + 2, // 每次递增的值
    value => value * value // 产生的结果
// 使用generate四个参数分别对应了for循环中的不同表达式，其中除了第一个参数是一个值外，其余三个参数都是函数，应该保持着三个参数都是纯函数。
)
```

#### repeat 重复数据的数据流
```js
const source$ = Observable.of(1, 2, 3);
const repeated$ source$.repeat(10); // 将source$的数据产生10遍，共产生30个数据，每次循环都会经历订阅，退订的过程
```

#### 三个极简的操作符：empty never throw 
```js
const source$ = Observable.empty(); // 直接产生完结对象，没有参数，不产生任何数据
const source$ = Observable.throw(new Error('直接报错')); // 直接抛出错误，立即完结
const source$ = Observable.nerver(); // nerver 真的什么都不做，即不吐出数据，也不完结，也不产生错误，就这样待着，直到天荒地老
```
### 创建异步数据的Observable对象
#### interval timer 定时产生数据
```js 
const source$ = Observable.interval(1000); // 从0开始，每秒产生一个数据，第一数据产生在订阅1秒后
// 如果想从1开始
const result$ = source%.map(x => x + 1); // 与map组合

// timer 指定产生第一数据的延迟时间
const source$ = Observable.timer(2000); // 2秒后产生第一个数据0，然后立刻完结
const source$ = Observable.timer(2000, 1000); // 2秒后产生第一个数据0，然后接下来每隔1秒依次产生一个数据
```

#### from 可以把一切转化为Observable
```js
const source$ = Observable.from([1, 2, 3]); // 依次输出1 2 3 甚至参数可以是argument

function * generateNumber(max) {
    for(let i = 0; i <= max; ++i) {
        yield i;
    }
}
const source$ Observable.from(generateNumber(3)); // 甚至可以消化yield

const source$ = Observable.from('abc'); // 输出 a b c 
```

#### fromPromise 异步处理交接
```js
const promise = Promise.resolve('good');
const source$ = Observable.from(promise);
source$.subscribe(
    console.log,
    error => console.log('catch', error),
    () => console.log('complete')
)
```

#### fromEvent 把dom中的事件转化为Observable对象中的数据
```js
// 第一个参数是一个事件源，如dom元素，第二个参数是事件名称，对应dom事件就是click,mousemove这样的字符串
const event$ = Rx.Observable.fromEvent(document.querySelector('#clickMe'), 'click');
event$.subscribe(
    e => {
        // ...
    }
)
// 还可以使用node的events中获得数据
import EventEmitter from 'events';
const emitter = new EventMitter();
const source$ = Observable.from(emitter, 'msg');
source$.subscribe(
    console.log
);

emitter.emit('msg', 1);
eemitter.emit('msg', 2);
mitter.emit('another-mag', 3); // 别的事件
emitter.emit('msg', 4);
// 输出 1 2 4
```

#### fromEventPattern 
```js
// fromEventPattern 接受2个函数参数，分别对应产生的Observable对象被订阅和退订时的动作,因为这两个参数是函数，具体动作可以任意定义，所以可以非常灵活
const emitter = new EventMitter();
const addHandler = handler => {
    emitter.addListener('msg', handler);
}
const removeHandler = handler => {
    emitter.removeListener('msg', handler);
}
const source$ = Observable.fromEventPattern(addHandler, removeHandler);
conset subscription = source$.subscirbe( // 订阅
    () => {}
)
subscription.unsubscribe() // 退订
// 其中参数handler相当于Observable的next函数
```

#### defer 推迟占用资源的一种惯用模式
```js
// defer 推迟占用资源的一种惯用模式
// defer 接受一个函数作为参数，当defer产生的Observable对象被订阅的时候，defer的函数参数就会被调用，预期这个函数会返回另一个Observable对象，也就是defer转嫁所有工作的对象，因为Promise和Observable的关系，defer也很贴心的支持返回Promise对象的函数参数。
const observableFactory = () => Observable.of(1, 2, 3);
const source$ = Observable.defer(observaleFactory);
```

## 第二大类：合并数据操作符

| 操作符 | 功能 |
| ------ | ------ |
| concat concatAll | 把多个数据以首尾相连方式合并 |
| merge mergeAll | 把多个数据流中数据以先到先得方式合并 |
| zip zipAll | 把多个数据流中最新产生的数据以一一对应方式合并 |
| combineLatest combineAll withLatestFrom | 持续合并多个数据流中最新产生的数据 |
| race | 从多个数据流中选取第一个产生内容的数据流 |
| startWith | 在数据流前面添加一个指定数据 |
| forkJoin | 只获取多个数据流最后产生的那个数据 |
| switch exhaust | 从高阶数据流中切换数据源 |

例子：
#### concat 首尾相连
```js
// concat既有实例操作符，也有静态操作符方式
const source1$ = Observable.of(1, 2, 3);
const source2$ = Observable.of(4, 5, 6);
const concated$ = source1$.concat(source2$);
// 还可以
const concated$ = Observable.concat(source1$, source2$);
/** 
concat 的工作方式
1 从第一个Observable 对象获取数据，把数据传给下游
2 当第一个Observable 对象 complete 后，concat就会去第二个Observable对象获取数据
3 依次类推 直到最后一个Observable完结之后，concat产生的Observable也就完结了
注意：
concat 会在第一个Observable对象完结后去订阅下一个Observable对象，所以参与到conct的Observable对象应该都能完结。如果其中有一个Observable对象不会完结，那么后面的Observable对象永远都没有上场的机会
*/
```

#### merge 先到先得快速通过
```js
/**
merge 与concat不同，merge会第一事件订阅所有的上游Observable,然后对上游的数据采取‘先到先得’的策略，任何一个Observale只要有数据推下来，就立刻转给下游Observable对象。
merge 在第一时间就订阅上游的所有Observable对象，所以某个上游数据永不完结，也不影响其他的Observable对象
*/
const source1$ = Observable.timer(0, 1000).map(x => x+"A");
const source2$ = Observable.timer(500, 1000).map(x => x+"B");
const merged$ = Observable.merge(source1$, source2$);
merged$.subscribe(
    console.log,
    null,
    () => console.log('complete')
)
// 以上代码会每隔500毫秒输出一行结果，永不停歇
// 0A 0B 1A 1B 2A 2B ...

// 同步限流
// merge 可以有一个可选参数concurent,用于指定可以同时合并的Observable对象的个数
const source$1 = Observable.timer(0, 1000).map(x => x+"A");
const source$2 = Observable.timer(0, 1000).map(x => x+"B");
const source$3 = Observable.timer(0, 1000).map(x => x+"C");
const merged$ = source$1.merge(source$2, source$3);
```

#### zip:拉链式组合
```js
// zip 要像拉链一样做到一对一咬合
// 只要任何一个上游的Observable对象完结，zip只要给这个完结的Observable对象吐出所有数据找到配对的数据，那么zip就会给下游一个complete信号
const source$1 = Observable.interval(1000);
const source$2 = Observable.of('a', 'b', 'c');
const ziped$ = Observable.zip(source$1, source$2);
// 输出：
[0, 'a']
[1, 'b']
[2, 'c']
// complete

// 数据积压问题
// 可以使用节流(throttle)，防抖(debounce) 相关系列的操作符来抛弃不需要的值
```

#### combineLatest 合并最后一个数据
```js
// 当任何一个上游Observable产生数时，从所有输入Observable对象中拿最后一次产生的数据（最新数据）,然后把这些数据组合起来传给下游。注意，这种方式和zip不一样，zip对上游数据只使用一次，用过一个数据之后就不会再用，但是combineLatest可能会反复使用上游产生的最新数据，只要上游不产生新的数据，那combineLatest就会反复使用这个上游最后一次产生的数据
// 只要当所有上游Observable都完结后，combineLatest才会给下游一个complete信号，表示不会有任何数据更新了

// 定制下游数据
// combinLatest的最后一个参数可以是一个函数，这里我们称为project,project的作用是让combineLatest把所有上游数据的最新数据扔给下游之前做一下组个处理，这样就可以不用传递一个数组下去，可以传递任何由最新数据产生的对象
const source$1 = Observable.interval(1000)
const source$2= Observable.timer(0, 1000)
const project = (a, b) => `${a} and ${b}`
const result$ source1$.combineLatest(source2$, project)

// 多重依赖问题
const origin$ = Observable.timer(0, 1000)
const source$1 = origin$.map(x => x+"A")
const source$1 = origin$.map(x => x+"B")
const result$ = source1$.combineLatest(source$2);
// 这里source$1和source$2会被origin$同时触发， 同一时间内应该只得到1个输出才对，但是现实是得到2个
// 1秒后
['1a', '0b']
['1a', '1b']
// 2秒后
['2a', '1b']
['2a', '2b']
```

#### withLatestFrom 类似于 combineLatest,但是上游只能由一个Observable对象驱动
```js
// withLatestFrom只有实例操作符的形式(因为：使用此操作符时，Observable对象地位不对等了)
const source$1 = Observable.interval(2000)
const source$2= Observable.timer(0, 1000)
const result$ = source$1.withLatestFrom(source$2, (a, b) => a+b);
// 作为参数的Observable对象只能贡献数据，不能控制产生数据的时机，节奏
// 每隔2秒输出一行
// 1
// 4
// 7
// 10

// withLatestFrom combineLatest 的选择：
// 如果要合并完全独立的Observable对象，使用combineLatest
// 如果要把一个Observable对象‘映射’新的数据流，同时要从其他Observable对象获取最新数据，那么使用withLatestFrom
```

#### race 胜者通吃
```js
// rece就是竞争，多个Observable对象在一起，看谁先产生数据，不过这种竞争是十分残酷的，胜者通吃，败者则市区所有机会
const source$1 = Observable.interval(2000)
const source$2 = Observable.timer(0, 1000)
const result$ = source$1.race(source$2);
// 如果race确定了胜利者，那么就会退订其他输入的Observable对象
```

#### startWith
```js
// startWith只要实例操作符的形式，其功能是让一个Observable对象在被订阅的时候总是先吐出指定的若干数据，
const source$1 = Observable.timer(0, 1000)
const result$ = source$1.startWith('start'); // 支持多个参数，同步输出
// 输出
// start 立刻输出
// 0 1秒后
// 1 2秒后
```

#### forkJoin ~== Promise.all
```js
// forkJoin只要静态操作符，接受多个Observable对象作为参数，它只会产生一个数据，因为他会等待所有参数Observable对象对象的最后一个数据，
// 也就是说，只要当所有Observable都完结，forkJoin会把所有输入Observable对象产生的最后一个数据合并成下游唯一的数据
// 所以说forkJoin就是rxjs界的Promise.all
const source$1 = Observable.interval(2000)
const source$2 = Observable.timer(0, 1000)
const result$ = source$1.forkJoin(source$2);
```

#### 高阶Observable 高阶数据流
```js
// 高阶函数就是产生函数的函数：参数为函数，返回也是函数
// 简单的高阶Observable
const hot$ = Observable.interval(1000).take(2).map(x => Observable.interval(1500).map(y => x+':'+y).take(2));
// 注意：高阶Observable完结，不代表内部Observable完结，但是内部Observable却不会岁随主干Observable的完结而完结，因为作为独立Observable，他们有自己的生命周期

// 高阶Observable的意义
// 高阶Observable的本质是用管理数据的方式来管理多个Observable对象，它的存在意义就在与此
```

#### 操作高阶Observable的合并类操作符

* concatAll
* merageAll
* zipAll
* combineAll(例外)

all代表全部，这些操作符功能有差异，但都是把一个高阶Observable的所有的内部Observable组合起来，所有这类操作符全部都只有实力操作符的形式

#### concatAll 
```js
// concatAll 只有一个上游Observable对象，这个observable对象预期是一个高阶Observable对象，concatAll会对内部的Observable对象做concat操作
const ho$ = Observable.interval(1000)
            .take(2)
            .map(x => Observable.interval(1500).map(y => x+':'+y).take(2));
const concated$ = ho$.concatAll();
// 0:0 # 开始订阅第一个内部Observable
// 0:1 
// 1:0 # 开始订阅第二个内部Observable
// 1:1
// concatAll 首先会订阅上游产生的第一个内部Observable对象，抽取其中的数据，然后只有当第一个Observable对象完结的时候，才会去订阅第二个Observable对象。
// 也就是说，虽然高阶Observable对象已经产生了第二个Observable对象，不代表concatAll会立刻去订阅它，因为和这个Observable对象是懒执行，所以不去订阅自然也不会产生数据。
```

#### mergeAll
```js
// mergeAll 就是处理高阶Observable的merge,
const ho$ = Observable.interval(1000)
            .take(2)
            .map(x => Observable.interval(1500).map(y => x+':'+y).take(2));
const concated$ = ho$.margeAll();
// mergeAll 对内部的Observable的订阅策略和concatAll不同，mergeAll只要发现上游产生一个内部Observable就会立刻订阅，并从中抽取数据
```

#### zipAll
```js
const ho$ = Observable.interval(1000)
            .take(2)
            .map(x => Observable.interval(1500).map(y => x+':'+y).take(2));
const concated$ = ho$.zipAll();
// ["0:0", "1:0"]
// ["0:1", "1:1"]

const ho$ = Observable.interval(1000).take(2).concat(Observable.never()) // 形成了只有两个数据的永不完结的数据流
            .map(x => Observable.interval(1500).map(y => x+':'+y).take(2));
const concated$ = ho$.zipAll();
// 现在zipAll的上游是一个永不完结的Observable，当它拿到2个内部Observable的时候，无法确定是不是还有新的内部Observable产生，而根据拉链的工作方式，来自不同数据源的数据要一对一配对，
// 这样一来，zipAll就只能等待，等待上游高阶Observable完结，这样才能确定内部Observable对象的数量，如果上游的高阶Observable不完结，那么zipAll 就不会开始工作
```

#### switch 切换输入Observable
```js
// switch 的含义就是 切换，总是切换到最新的内部Observable对象获取数据，每当switch的上游高阶Observable产生一个内部Observable对象，switch都会立刻订阅最新的内部Observable对象上
// 如果已经订阅了之前的内部Observable对象，就会退订那个多事的内部Observable对象，这个 用上新的，舍弃旧的 动作，就是切换
const ho$ = Observable.interval(1000)
            .take(2)
            .map(x => Observable.interval(1500).map(y => x+':'+y).take(2));
const result$ = ho$.switch();
// 1:0
// 1:1
// switch 首先订阅了第一个内部Observable对象，但是这个内部对象还没来得及产生第一个数据0:0(1000ms + 1500ms 后)，第二个内部Observable对象就产生了(100ms + 1000ms 后)，
// 这样switch就会切换动作，切换到第二个内部Observable上，因为之后没有新的内部Observable对象产生了，switch就会一直从第二个内部Observable对象获取数据，于是最后得到的数据就是 1:0,1:1

// 在上游Observable产生新的内部Observable时进行切换。订阅的是内部的Observable

// 注意和 race 不同，race 是别人没有机会，而switch可以翻来覆去一直抢

// 完结条件：1 上游高阶Observable已经完结; 2 当前内部Observable已经完结
```

#### exhaust 耗尽
```js
// 在耗尽当前内部Observable的数据之前不会切换到下一个内部Observable对象
const ho$ = Observable.interval(1000)
            .take(3)
            .map(x => Observable.interval(700).map(y => x+':'+y).take(2));
const result$ = ho$.exhaust();
// exhaust 首先从第一个内部Observable对象获取数据，然后再考虑后续的内部Observable对象
// 第二个内部Observable生不逢时，当它产生的时候第一个内部Observable对象还没有完结，这时候exhaust会直接忽略第二个Observable对象，甚至不会去订阅它，、
// 第三个内部Observable对象会被订阅病提取数据，是因为在它出现之前，第一个内部Observable都已经完结了

// 完结条件：1 上游Observable对象完结; 2 最新的内部Observable对象完结
```

## 第三大类：辅助类操作符

| 操作符 | 功能 |
| ------ | ------ |
| count | 统计数据流中产生的所有数据个数 |
| max min | 获得数据流中的最大和最小数据 |
| Reduce | 对数据流中所有数据进行规约操作 |
| every | 判断一个数据流是否不包含任何数据 |
| find findIndex | 找到第一个满足判定条件的数据 |
| isEmpty | 判断一个数据流是否不包含任何数据 |
| defaultEmpty | 如果一个数据流为空就默认产生一个指定的数据 |

例子：