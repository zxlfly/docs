# Vue Test Utils
一般会结合这个一起使用

## 常见操作
- 触发input键盘事件
  - ``input.element.dispatchEvent(new KeyboardEvent('keydown', { key: 'Escape' }))``
  - ``input.trigger('keydown', {code: Escape,})``
- 滚动
```
const wait = (delay = 300) =>new Promise(resolve => setTimeout(() => resolve(true), delay))
const makeScroll = async (
  dom: Element,
  name: 'scrollTop' | 'scrollLeft',
  offset: number
) => {
  const eventTarget = dom === document.documentElement ? window : dom
  dom[name] = offset
  const evt = new CustomEvent('scroll', {
     detail: {
      target: {
       [name]: offset,
      },
    },
  })
  eventTarget.dispatchEvent(evt)
  // must use setTimeout instead of nextTick to wait dom change
  return await wait()
}
await makeScroll(scrollDom, 'scrollTop', 100)
```
- 回调函数触发
  - ``const selectValueCB = jest.fn()``