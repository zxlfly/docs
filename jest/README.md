# Jest默认支持的是CommonJS规范，不支持es6的方式
可以使用babel转换实现支持  
安装``@babel/core``和``@babel/preset-env``  
配置``.babelrc``
```
{
    "presets":[
        [
                "@babel/preset-env",{
                "targets":{
                    "node":"current"
                }
            }
        ]
    ]
}
```
jest里面有一个babel-jest组件，它会做相应的检测

# jest配置
``npx jest --init``生成配置文件  
- coverageDirectroy
  - 代码测试覆盖率，生成一个coverage的文件夹，里面有测试结果的图表
  - ``coverageDirectory : "coverage"   //打开测试覆盖率选项``
    - 可以不使用coverage为名称，换其他的也行，会改变存放结果文件的文件夹的名称
  - ``npx jest --coverage``npm命令，也可以配置到script中

# [jest中的匹配器](https://jestjs.io/docs/zh-Hans/expect)
有很多具体的以文档为主，这里会介绍几个常见的。示例在matchers.test.js
- toBe()：严格相等相当于===
- toEqual()：内容相等，就可以通过测试
- toBeNull()：只匹配null值
- toBeUndefined()：匹配undefined
- toBeDefined()：只要定义过了，都可以匹配成功
- toBeTruthy()：真假匹配器，真的时候可以通过
- toBeFalsy()：真假匹配器，假的时候可以通过
- toBeGreaterThan()：大于传入的数值，就可以通过测试
- toBeLessThan()：少于一个数字时，就可以通过测试。
- toBeGreaterThanOrEqual()：大于等于期待数字时，可以通过测试
- toBeLessThanOrEqual()：小于等于期待数字时，可以通过测试
- toBeCloseTo()：消除JavaScript浮点精度错误的匹配器
- toMatch()：字符串包含匹配器
- toContain()：数组的匹配器(set也可以)
- toThrow()：异常进行处理的匹配器
- not：相反，取反

# 异步代码的测试方式
asynchronous.test.js
- 回调函数的方式，需要加入一个done方法
- promise方式，需要return
- async...await...
- 在处理一些模拟等待过程的时候可以使用定时模拟函数进行快进
  - ``jest.useFakeTimers ... jest.runAllTimers``

# 钩子函数
- beforeAll()钩子函数的意思是在所有测试用例之前进行执行
- afterAll()钩子函数是在完成所有测试用例之后才执行的函数。
- beforeEach()钩子函数，是在每个测试用例前都会执行一次的钩子函数
- afterEach()钩子函数，是在每次测试用例完成测试之后执行一次的钩子函数

# 对测试用例进行分组
最原始的方式是分成多个文件，也可以使用``describe()``
```
describe('组名',()=>{...})
```

# 钩子函数作用域
- 钩子函数在父级分组可作用域子集，类似继承
- 钩子函数同级分组作用域互不干扰，各起作用
- 先执行外部的钩子函数，再执行内部的钩子函数
  
# mock
```
cpnst {getData} = require(....)
cpnst axios = require('axios')
jest.mock('axios')
if('fetch test',async ()=>{
  axios.get.mockResolvedValueOnce('123')
  axios.get.mockResolvedValueOnce('456')
  const data1 = await getData()
  const data2 = await getData()
  expect(data1).toBe('123')
  expect(data2).toBe('456')
})
```