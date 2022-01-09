# Rxjs
理解Rx的关键是把任何变化想象成事件流
```
// 输入 "hello world"
var input = Rx.Observable.fromEvent(document.querySelector('input'), 'input');

// 过滤掉小于3个字符长度的目标值
input.filter(event => event.target.value.length > 2)
  .map(event => event.target.value)
  .subscribe(value => console.log(value)); // "hel"

// 延迟事件
input.delay(200)
  .map(event => event.target.value)
  .subscribe(value => console.log(value)); // "h" -200ms-> "e" -200ms-> "l" ...

// 每200ms只能通过一个事件
input.throttleTime(200)
  .map(event => event.target.value)
  .subscribe(value => console.log(value)); // "h" -200ms-> "w"

// 停止输入后200ms方能通过最新的那个事件
input.debounceTime(200)
  .map(event => event.target.value)
  .subscribe(value => console.log(value)); // "o" -200ms-> "d"

// 在3次事件后停止事件流
input.take(3)
  .map(event => event.target.value)
  .subscribe(value => console.log(value)); // "hel"

// 直到其他 observable 触发事件才停止事件流
var stopStream = Rx.Observable.fromEvent(document.querySelector('button'), 'click');
input.takeUntil(stopStream)
  .map(event => event.target.value)
  .subscribe(value => console.log(value)); // "hello" (点击才能看到)
```
## 常见创建类操作符
- from：可以把数组、promise、Iterable转化为Observable
- fromEvent：可以把事件转化为Observable
  - 接受参数dom和事件
- of：接受一系列的数据，并把它们emit出去
  - 可以用来操作对象，转换成流，对象的结构可以不同
  - 也可以处理数组，但是from更方便
- Interval
  - 循环执行
- Timer
  - 可以设置延迟时间，循环执行
  - 可以只执行一次

## Observable
- 三种状态：
  - next
    - 正常操作
  - error
    - 错误
  - complete  
    - 结束
- 特殊的：
  - 永不结束
    - 没有complete，例如一个计时器每隔一秒发射一个状态
  - Never
    - 既不发射也不结束也没有元素，就是空的
    - 多用于测试
  - Empty（结束但不发射）
    - 和Never相比直接进入complete
    - 多用于测试
  - Throw
    - 和Never相比直接进入error

## 常见操作符
- map
  - 和js里面的用起来一样
- mapTo
  - .naoTo(any)
  - 这种就是不关心值，只要事件发生了就行
- pluck
  - 对map的简化操作
  - map接受到event
  - pluck接受的就是event.target

## 常见工具操作符：do
流事件执行前执行，和外部交互的桥梁，和subscribe类似接受三个参数
## 常见变换类操作符：scan
接受函数为参数，会记住之前的结果，下次可以使用
## 常见数字类操作符：reduce
和上面一样，不过需要计算的是最终值，所以需要注意是否是循环执行下去的
## 过滤类操作符
- filter
- take
  - 取几个
- first/last
  - last使用的时候要注意，是否是循环执行下去的
- skip
  - 跳过几个
- debounce
  - 根据时间间隔过滤
  - 延时发送源 Observable 发出的值,但如果源 Observable 发出了新值 的话，它会丢弃掉前一个等待中的延迟发送。
- debounceTime
  - 根据时间间隔过滤
  - 延时发送源 Observable 发送的值,如果源 Observable 又发出新值，会丢弃正在排队的发送。
  -  该操作符会追踪源 Observable 的最新值, 并且发出它当且仅当在 dueTime 时间段内 没有发送行为。
  -  如果新的值在dueTime静默时间段出现, 之前的值会被丢弃并且不会在输出 Observable 中发出。
- distinct
  - 保留不一样的
  - 所有的整个序列
- distinctUntilChanged
  - 保留不一样的
  - 之和前一个比

## 合并操作符
- merge
  - 按照时间顺序合并
- concat
  - 拼接
  - 一个流接着一个流
  - 注意无穷序列
- startWith
  - 设置一个开始值
  - 初始值
- combineLatest
  - 只有有新的元素出现，就会产生新流中对应的元素
- withLatestFrom
  - 以主流为主
  - 主流产生新元素，就去其他流取最新值
  - 产生新流中对应的元素
- zip
  - 必须都出现了新元素，才会产生新流中对应的元素