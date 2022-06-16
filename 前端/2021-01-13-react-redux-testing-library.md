---
layout: post
title: '前端单元测试实战：React + Redux Testing Library'
categories: [前端]
tags: [Tutorial, UnitTest, CICD, TestingLibrary, React, Redux, Router, CQRS, TDD]
published: True
---

> PPT 内容: https://jimmylv.gitee.io/slides/react-test

# 为什么要有单元测试？

## 走 🚶 vs 🏃 跑

![img](https://jimmylv.gitee.io/slides/images/busy.jpeg)

> **写不好是能力问题，不写则是态度问题。**

单元测试客观上可以让开发者的工作更高效，React 应用的单元测试是一定要的。

## 单元测试的上下文

![image-20210228182332955](https://raw.staticdn.net/JimmyLv/images/master/2021/image-20210228182332955.png)

谈任何东西都一定要有个上下文。你的论述不能是「因为单元测试有这些好处，所以我们要做单元测试」，而应该是「不做单元测试我们会遇到什么问题」，这样才能回答「为什么要写单元测试」的问题。那么我们谈论单元测试的上下文是什么呢？不做单元测试我们会遇到什么问题呢？上图为一个产品从 idea 分析、设计、开发、测试到交付并获取市场反馈的过程。

## 缩短反馈周期

![img](https://jimmylv.gitee.io/slides/images/tdd/xp-feedback.png)

而**单元测试的上下文就是存在于「敏捷」当中**。敏捷为的是更快地交付有价值的可工作的软件。为此，它有一个指标来度量这个「更快」，那就是 lead time，它度量的是一个 idea 从提出被验证，到最终上生产环境面对用户的时间。显然，这个时间越短，软件获得反馈的时间就越短，对价值的验证就越快发生。

## 单元测试的意义

- 如果你说我的业务部门不需要频繁上线，并且我有足够的人力来覆盖手工测试，那你可以不用单元测试
- 如果你说我不在意代码腐化，并且我也不做重构，那你可以不用单元测试
- 如果你说我不在意代码质量，好几个没有测试保护的 if-else 裸奔也不在话下，脑不好还做什么程序员，那你可以不用单元测试
- 如果你说我确有快速部署的需求，但我们不 care 质量问题，出回归问题就修，那你可以不用单元测试

除此之外，你就需要写单元测试。如果你想随时整理重构代码，那么你需要写单元测试；

如果你想有自动化的测试套件来帮你快速验证提交的完整性，那么你需要写单元测试。

这个结论对我们写不写单元测试有什么影响呢？答案是，不写单元测试，你就快不起来。为啥呢？因为每次发布，你都要投入人力来进行手工测试；因为没有测试，你倾向于不敢随意重构，这又导致代码逐渐腐化，复杂度使得你的开发速度降低。

那么在这个上下文中来谈要不要单元测试，我们就可以很有根据了，而不是“开发爽了就用，不爽就不用”这样含糊的答案

## 单元测试与自动化的关系

![img](https://raw.staticdn.net/JimmyLv/images/master/2018/20181029222614.png)

**自动化**回答的是**要不要自动化的单元测试**这个问题。测试是重构的唯一保障，也就是说，没有测试，基本上就没法重构代码（重构指的是 [不改变软件可观测行为的前提下改善代码内部设计或实现](https://www.martinfowler.com/bliki/DefinitionOfRefactoring.html) ），基本上就只能看着代码腐化。那么，基本上只要你的系统需要持续发展，你就需要单元测试。

**反馈速度**回答的是**要不要 TDD、测试先行还是后补**这个问题。答案是，需要 TDD，最好先行，因为[可以提高反馈速度](https://github.com/linesh-simplicity/linesh-simplicity.github.io/issues/197)。

# 单元测试基础

## 测试框架 - Jest

- Fast 天下武功，唯快不破。
- Opinionated 开箱即用。
- Watch Mode 守护模式。
- Snapshot Testing 快照测试。

## 第一个 Jest 实例

```bashjs
mkdir jest-demo && cd \$_
yarn init -y #--yes
yarn add jest -D #--de
```

首先创建 jest-demo 项目并安装 jest 作为项目 devDependencies 依赖：

## 一个简单的纯函数

```js
const sum = (a, b) => a + b

module.exports = { sum }
```

然后创建一个 math.js 文件，输入一个我们稍后测试的 sum 函数:

## 麻雀虽小五脏俱全

```js
const { sum } = require('./math')

describe('Math module', () => {
  test('should return sum result when one number plus another number', () => {
    // Given
    const number = 1
    const anotherNumber = 2
    // When
    const result = sum(number, anotherNumber)
    // Then
    expect(result).toBe(2)
  })
})
```

接下来，让我们写第一个测试。在同一个文件夹中创建一个 math.test.js 文件，在这里我们将使用 Jest 来测试 math.js 中定义的函数:

![image-20210228183001013](https://raw.staticdn.net/JimmyLv/images/master/2021/image-20210228183001013.png)

然后运行 yarn test （添加 NPM Script）你就可以看到相应的结果。

## Given/When/Then 的套路

![image-20210228183013307](https://raw.staticdn.net/JimmyLv/images/master/2021/image-20210228183013307.png)

麻雀虽小五脏俱全，在上面的例子当中，我们可以看到很多的测试元素，下面将会一一介绍：

首先我们看到的是一个由 it 包裹的测试主体最小单元，采用了 Given When Then 的经典格式，我们常常称之为测试三部曲，也可以解释为 3A 即：

```js
expect(1 + 1).toBe(2)
expect(1 + 1).not.toBe(3)
```

在 expect 后面的 toBe 称之为 Matcher，是断言时的判断语句以验证正确性 ✅，在后面的文章中我们还会接触更多 Matchers，甚至可以扩展一些特别定制的 Matchers。

![image-20210228183057426](/Users/Jing/Library/Application Support/typora-user-images/image-20210228183057426.png)

修改断言的结果，就可以看到成功后的结果了：

# 模块间依赖 Fake/Stub/Mock/Spy

![image-20210228183104589](https://raw.staticdn.net/JimmyLv/images/master/2021/image-20210228183104589.png)

如同人类世界中的羁绊，软件模块之间必然也免不了依赖。[Martin Fowler](https://martinfowler.com/) 在 [UnitTest](https://martinfowler.com/bliki/UnitTest.html) 这篇文章当中将单元测试作了一个重要的区分，即你所测试的单位应该是社交型（Social Tests）还是独立型（Solitary Tests）？ 想象一下你正在测试一个 Order Class 的 price() 方法，而 price() 方法需要在 Product 和 Customer Class 中调用一些函数。如果你希望单元测试所测试的 Order 模块是独立的，那么你就不想直接使用真正的 Product 或 Customer Class，因为 Customer Class 的错误会直接导致 Order Class 的单元测试失败。相反，你可能会使用一个替身作为依赖的对象，也就是我们接下来会提到的 Fake/Stub/Mock/Spy。

- Database 数据库
- Network requests 网络请求
- access to Files 存取文件
- any External system 任何外部系统

其实在 Jest 当中，Fake/Stub/Mock/Spy 这些概念或许会有所混淆，而这跟 JavaScript 语言本身的特点有一定关系，但是我觉得 Jest 通过统一的 fn() 方法把问题解决得还比较恰当，让我们来一块儿看看实例 🌰：

## Mock 用于替代整个模块

```js
import SoundPlayer from './sound-player'

const mockPlaySoundFile = jest.fn()

jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return { playSoundFile: mockPlaySoundFile }
  })
})
```

我们可以看到 jest.mock() 方法中的第二个参数是一个函数，那么我们就可以完全接管整个 ./sound-player JavaScript 模块，比如说这里的 playSoundFile 本来应该是从 ./sound-player 这个文件当中 export 出来的，而被 Mock 之后我们的测试就可以使用 Mock 所返回的数据或方法，从而保证模块所返回的内容是我们所期望的。但这时需要注意的是，该模板的所有功能都已经被 Mock 掉，而不会再从原模块当中返回，所以我们就需要重新实现该模块中的所有功能。可别一不小心就成了张艺谋导演《影》片中的影子，被完全“取而代之”，连夫人也被 Mock 所吸引。

## Stub 用于模拟特定行为

```js
const mockFn = jest.fn()
mockFn()
expect(mockFn).toHaveBeenCalled()

// With a mock implementation:
const returnsTrue = jest.fn(() => true)
console.log(returnsTrue()) // true;
```

这里的特定行为也可以是没有行为，jest.fn() 代表着我就是一个 Stub（桩），“你来我就在这里，你走我也依然在这里，风雨无阻”。不需要什么输入输出，只要能在测试的时候验证到 Stub 被调用过就行，也就能够断言到某处代码被执行，从而确定代码被测试所覆盖。而另一种特定行为就是返回特定的数据，即 Stub 也可以根据输入模拟返回一种输出，作为某些模块的替身帮它演戏，比如“小鲜肉们”遇到要跳车啦、要卿卿我我（误）的时候就要找替身，“一二三四五六七八”连台词都不用背还需要配音。

## Spy 用于监听模块行为

> Spy packages without affecting the functions code

```js
const video = require('./video')

it('plays video', () => {
  const spy = jest.spyOn(video, 'play')
  const isPlaying = video.play()

  expect(spy).toHaveBeenCalled()
  expect(isPlaying).toBe(true)
})
```

Spy 并不会影响到原有模块的功能代码，而只是充当一个监护人的作用，“你可以继续我型我秀上课讲小话，但是老师会偷偷告诉你妈妈，看你放学后老妈不打断你的腿”。比如说上文中的 video 模块中的 play() 方法已经被 spy 过，那么之后 play() 方法只要被调用过，我们就能判断其是否执行，甚至执行的次数。

## 如何 Mock 全局的方法？

```js
window.matchMedia = jest.fn().mockImplementation((query) => {
  return {
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
  }
})
```

把全局的数据 Mock 掉很简单，只需要像 window.document.title = undefined 这样简单 Fake 赋值就很完美。而像 matchMedia 这样的方法在 jsdom 里面并没有被实现，这时候我们当然就需要去把它 Mock 掉，简单把要用到的一些对象属性赋值就好，总之不至于在运行时报错。

## 代码模块的易测性

保持单元测试独立性的同时，也是在促使你去思考什么样的模块才是符合「职责单一原则」的。

单元测试站在使用者的角度来使用该模块，而代码的易测性也就代表着代码的可维护性。

从上文的一些例子当中，我们也可以看到，不管是 Fake/Stub/Mock/Spy 最最重要的一个原则就是「简单」，因为我们是在写测试代码，而所依赖的模块就应该以最简单的形态展现出来，绝不要给 jest.fn() 编写过于哪怕一点点复杂的逻辑。如果这个模块有多种表现形态，那就把它分成测试单元进行多次 Mock，每个 it() 单元测试一定是针对于单个功能点进行测试的。

# 如何测试异步代码？

```js
navigator.geolocation.getCurrentPostion() // chrome API 异步获取当前位置
```

异步是 JavaScript 中绕不开的永恒话题，多亏了 ES6+ 高级语法所提供的多种优雅的异步代码方式，让我们写测试代码的方式也多了好多种。（逃

让我们先来看一下什么是异步请求，这里有一个通过 Chrome API 获取当前位置的实例，可想而知 Chrome 要根据 GPS 信号才能算出当前的经纬度，相当于从卫星 🛰 来回走了一遭，怎么不会异步（代表有延时，延迟返回）呢？

## Callback 回调函数

```js
it('the data is peanut butter', (done) => {
  function callback(data) {
    expect(data).toBe('peanut butter')
    done()
  }

  fetchData(callback)
})
```

这是最最普通的方式，也是各大框架都支持的一种写法， done() 作为异步代码结束的结束标志，从而让测试框架“知道”在结束时进行断言。但这种方式侵入性比较强，对测试语句不友好且违背了 Given/When/Then 的三段式套路，就像回调地狱一样的道理，如果让 done() 充斥着测试那么代码也就变得混乱。

## Promise 让爱 then() 到底

```js
it('the data is peanut butter', () => {
  expect.assertions(1)
  return fetchData().then((data) => {
    expect(data).toBe('peanut butter')
  })
})

expect(Promise.resolve('lemon')).resolves.toBe('lemon')

expect(Promise.reject(new Error('octopus'))).rejects.toThrow('octopus')
```

其实这种方式也好不到哪去，无非就是把 done() 方式换成了 then() 又一次充斥在整个 expect 当中，混乱了 When 和 Then 两种本该分开的时刻。但也有一个不错的点，可以通过 Promise 的 .resolve() 和 .reject() 方法使测试分别验证正常或异常的情况。

## Async/Await 让异步变得同步

```js
test('the data is peanut butter', async () => {
  // given
  const params = {}

  // when
  const data = await fetchData(params)

  // then
  expect(data).toBe('peanut butter')
})
```

Async/Await 语法糖在业务代码当中就特别好使了，好处不多说直接看得见：原本需要 done() 或 then() 的地方都不再混乱，又一次回归到了正常的 Given/When/Then 三段式套路，让测试代码变得非常清晰易读。唯一需要注意的是， 额外的 expect.assertions(number) 其实是验证在测试期间所调用的断言数量，这在测试多层异步代码时很有用，以确保实际调用回调中的断言次数。

# React 组件测试

## 组件化与 UI 测试

![img](https://raw.staticdn.net/JimmyLv/images/master/2018/20181030220153.png)

在组件化出现之前，我们都压根不谈 UI 的**单元**测试，哪怕是对于 UI 页面层级的测试来说都是一件非常困难的事情。其实**组件化并不全是为了复用，很多情况下也恰恰是为了分治**，从而我们可以分组件对 UI 页面进行开发，然后分别对其进行单元测试。

前端组件化已经让 UI 测试变得容易很多，每个组件都可以被简化为这样一个表达式，即 UI = f(data)，这个纯函数返回的只是一个描述 UI 组件应该是什么样子的虚拟 DOM，本质上就是一个树形的数据结构。给这个纯函数输入一些应用程序的状态，就会得到相应的 UI 描述的输出，这个过程不会去直接操作实际的 UI 元素，也不会产生所谓的副作用。

## React 组件树的测试

![ComponentsTree](https://raw.staticdn.net/JimmyLv/images/master/2018/20181030211115.png)

按理来说按照纯函数这样的思路，React 组件的测试应该很简单的说。但与此同时，对 UI 渲染的组件树进行测试依然存在一个问题，从下图中可以看出，越处于上层的组件，其复杂度必然会随之提高。对于最底层的子组件来说，我们可以很容易得将其进行渲染并测试其逻辑的正确与否，但对于较上层的父组件来说，通常来说就需要对其所包含的所有子组件都进行预先渲染，甚至于最上面的组件需要渲染出整个 UI 页面的真实 DOM 节点才能对其进行测试，这显然是不可取的。

在单元测试中，通常我们希望将重点放在作为独立单元进行测试的组件上，并避免间接断言其子组件的行为。此外，对于包含许多子组件的组件，整个 render 树会变得非常之大，而反复 render 所有的子组件可能会减慢单元测试的速度。

## 测试金字塔

> 为了维持金字塔形状，一个健康、快速、可维护的测试组合应该是这样的：写许多小而快的单元测试。适当写一些更粗粒度的测试，写很少高层次的端到端测试。注意不要让你的测试变成冰淇淋那样子，这对维护来说将是一个噩梦，并且跑一遍也需要太多时间。（via [测试金字塔实战 – ThoughtWorks 洞见]([https://insights.thoughtworks.cn/practical-test-pyramid/)）](https://insights.thoughtworks.cn/practical-test-pyramid/)

![img](https://raw.staticdn.net/JimmyLv/images/master/2018/20181030211424.png)

## UI 测试策略

- 编写不同粒度的测试
- 层次越高，你写的测试应该越少

# React 组件的渲染方式

## ❌ 浅渲染 shallowMount(component[, options]) => Wrapper

```js
import { shallow } from 'enzyme'

describe('Enzyme Shallow', () => {
  it('App should have three <Todo /> components', () => {
    const app = shallow(<App />)
    expect(app.find('Todo')).to.have.length(3)
  })
})
```

浅渲染在将一个组件作为一个单元进行测试的时候非常有用，可以确保你的测试不会去间接断言子组件的行为。shallowMount 方法就是 Shallow Rendering 的封装，shallowMount 跟 mount 类似返回 mounted 和 rendered React 组件的 Wrapper，但只会渲染出组件的第一层 DOM 结构，其嵌套的子组件不会被渲染出来，从而使得渲染的效率更高，单元测试的速度也会更快。

## 全量渲染 mount(component[, options]) => Wrapper

```js
import { mount } from 'enzyme'

describe('Enzyme Mount', () => {
  it('should delete Todo when click button', () => {
    const app = mount(<App />)
    const todoLength = app.find('li').length
    app.find('button.delete').at(0).simulate('click')
    expect(app.find('li').length).to.equal(todoLength - 1)
  })
})
```

mount 方法则会将 React 组件和所有子组件渲染为真实的 DOM 节点，特别是在你依赖真实的 DOM 结构必须存在的情况下，比如说按钮的点击事件。完全的 DOM 渲染需要在全局范围内提供完整的 DOM API， 这也就意味着 React Test Utils 依赖于浏览器环境。

从技术上讲，你可以在真实的浏览器中运行，但由于在不同平台上启动真实浏览器的复杂性，更建议使用 JSDOM 在虚拟浏览器环境中运行 Node 中的测试。推荐使用 mount 的方法是依赖于一个名为 jsdom 的库，它本质上是一个完全在 JavaScript 中实现的 headless 浏览器。

## Testing Library vs Enzyme

React Testing Library 的 API 明显优于 Enzyme，不至于陷入细节，是用于测试 React 应用的一大利器。前端 UI 组件测试的最佳实践，使得我们可以使用它来更有效地测试组件。

```js
import { screen, render } from '@testing-library/react'

test('should show h1 title', () => {
  // given
  render(<App />)

  // when
  const result = screen.getByTestId('title').textContent

  // then
  expect(result).toBe('Edmodo')
})
```

## [**Which query should I use?**](https://testing-library.com/docs/guide-which-query)

| **type**       | **_No Match_** | **_1 Match_** | **_1+ Match_** | **_Await?_** |
| -------------- | -------------- | ------------- | -------------- | ------------ |
| **_getBy_**    | _throw_        | _return_      | _throw_        | _No_         |
| **_findBy_**   | _throw_        | _return_      | _throw_        | _Yes_        |
| **_queryBy_**  | _null_         | _return_      | _throw_        | _No_         |
| **getAllBy**   | throw          | array         | array          | No           |
| **findAllBy**  | throw          | array         | array          | Yes          |
| **queryAllBy** | []             | array         | array          | No           |

## UI 交互行为的测试

![img](https://jimmylv.gitee.io/slides/images/react-user-event.png)

我们不但可以通过 find 方法查找 DOM 元素，还可以通过 simulate 方法在组件上模拟触发某个 DOM 事件，比如 Click，Change 等等。对于浅渲染来说，事件模拟并不会像真实环境中所预期的那样进行传播，因此我们必须在一个已经设置好了事件处理方法的实际节点上才能够调用，实际上 .simulate() 方法将会根据模拟的事件触发这个组件的 prop。例如，.simulate('click') 实际上会获取 对应的 clickHandler propsData 并调用它。

## ❌ Enzyme .find() 方法

```js
it('simulates click events', () => {
  // given
  const onButtonClick = sinon.spy()
  const wrapper = shallow(<Foo onButtonClick={onButtonClick} />)

  // when
  wrapper.find('button').simulate('click')

  // then
  expect(onButtonClick.calledOnce).to.be.true
})
```

## ✅ Testing Library userEvent

```js
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

test('should calculate result by input number', () => {
  render(<App />)

  userEvent.type(screen.getByPlaceholderText(/Please input your number/i), 15)

  expect(wrapper.getByTestId('number-result').textContent).toBe('FizzBuzz')
})
```

# 如何理解 Redux 模式？

## Redux 的前车之鉴

![Redux data flow diagram](https://redux.js.org/assets/images/ReduxDataFlowDiagram-49fa8c3968371d9ef6f2a1486bd40a26.gif)

> Redux 是一个专为 React.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

古人说「读史让人明智」，学习历史是为了更好得前行，为了能够认识现在，看清未来。让我们来看看 Redux 的历史，Redux 借鉴于 Redux，而 Redux 的实现构想则最初出身于 [Flux](http://facebook.github.io/flux/docs/overview.html) ，这是一个由 Facebook 为其应用所设计的应用程序架构。Flux 模式在 JavaScript 应用里像是找到了新家一样，但其实只是借鉴了**领域驱动设计** (DDD) 和**命令-查询职责分离** (CQRS)。

## MVC 与 Flux 架构

![image-20210228183300367](https://raw.staticdn.net/JimmyLv/images/master/2021/image-20210228183300367.png)描述 Flux 最普遍的一种的方式就是将其与 **Model-View-Controller** (MVC) 架构进行对比。

在 MVC 当中，一个 Model 可以被多个 Views 读取，并且可以被多个 Controllers 进行更新。在大型应用当中，单个 Model 会导致多个 Views 去通知 Controllers，并可能触发更多的 Model 更新，这样结果就会变得非常复杂。

## CQRS 与 Flux 架构

![image-20210228183307490](https://raw.staticdn.net/JimmyLv/images/master/2021/image-20210228183307490.png)

而 Flux 以及我们要学习的 Redux 则是试图通过强制单向数据流来解决这个复杂度。在这种架构当中，Views 查询 Stores（而不是 Models），并且用户交互将会触发 Actions，Actions 则会被提交到一个集中的 Dispatcher 当中。当 Actions 被派发之后，Stores 将会随之更新自己并且通知 Views 进行修改。这些 Store 当中的修改会进一步促使 Views 查询新的数据。

MVC 和 Flux 最大的不同就是查询和更新的分离。在 MVC 中，Model 同时可以被 Controller 更新*并且*被 View 所查询。在 Flux 里，View 从 Store 获取的数据是只读的。而 Stores 只能通过 Actions 被更新，这就会影响 Store 本身*而不是*那些只读的数据。

## CQRS 命令-查询职责分离

1. 如果一个方法修改了这个对象的状态，那就是一个 _command_（命令），并且一定不能返回值。
2. 如果一个方法返回了一些值，那就是一个 _query_（查询），并且一定不能修改状态。

![Redux data flow diagram](https://redux.js.org/assets/images/ReduxDataFlowDiagram-49fa8c3968371d9ef6f2a1486bd40a26.gif)

# Redux 背后的基本思想

所以说， Redux 就是把组件的共享状态 “state” 抽取出来，以**一个**全局 “store” 的单例模式统一管理。

在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为。

## Redux 三大原则：强制遵守一定的规则

### 1. 单一数据源

整个应用的 [state](https://cn.redux.js.org/docs/Glossary.html#state) 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 [store](https://cn.redux.js.org/docs/Glossary.html#store) 中。任何组件都能直接获取 store 状态，这也就是 CQRS 中 _query_（查询）的一种实现。

### 2. State 是只读的

唯一改变 state 的方法就是显式地触发一个 [action](https://cn.redux.js.org/docs/Glossary.html#action)，action 是一个用于描述已发生事件的普通对象。如果需要支持异步数据流，则需要像 [redux-thunk](https://github.com/gaearon/redux-thunk) 或 [redux-promise](https://github.com/acdlite/redux-promise) 这样支持异步的 middleware 中间件。

### 3. 使用纯函数来执行修改

为了描述 action 如何改变 state tree ，你需要编写 [reducers](https://cn.redux.js.org/docs/Glossary.html#reducer)。这也就是 CQRS 中 _command_（命令）的一种实现。

# 如何对 Redux 进行单元测试

得益于 Redux 能够将 React 应用的共享状态进行隔离，我们的代码也因此变得更加结构化且易于维护，Redux 中的 reducer、action 和 selector 都被放在了合理的位置，承担不同的职责 ，这也使得对它们进行单元测试变得容易很多。

## reducers 测试

```js
import reducers from './reducers'
import actions from './actions'

test(`
  should merge user comments and remove duplicated comments by comment id
  when action saveUserComments is dispatched with new fetched comments
`, () => {
  const state = {
    comments: [{ id: 1, content: 'comments-1' }],
  }
  const comments = [
    { id: 1, content: 'comments-1' },
    { id: 2, content: 'comments-2' },
  ]

  const expected = {
    comments: [
      { id: 1, content: 'comments-1' },
      { id: 2, content: 'comments-2' },
    ],
  }

  const result = reducers(state, actions.saveUserComments(comments))

  expect(result).toEqual(expected)
})
```

Reducer 很容易被测试，因为它们仅仅是一些完全依赖参数的函数。最为简单的 reducer 测试，仅一一对应保存数据切片。此种 reducer 可以不需要测试覆盖，因为基本由架构简单和逻辑简单保证，不需要靠读测试用例来理解。而一个较为复杂、具备测试价值的 reducer 在保存数据的同时，还可能进行了合并、去重等操作。

## actions 测试

```js
import * as actions from './actions'

test('should dispatch saveUserComments action with fetched user comments', () => {
  const comments = []
  const expected = {
    type: 'saveUserComments',
    payload: {
      comments: [],
    },
  }

  const result = actions.saveUserComments(comments)

  expect(result).toEqual(expected)
})
```

Action 应对起来略微棘手，因为它们可能需要调用外部的 API。当测试 action 的时候，我们需要增加一个 mocking 服务层——例如，我们可以把 API 调用抽象成服务，然后在测试文件中用 mock 服务响应所期望的 API 调用。

## 异步 Action 测试

```js
// product.test.js
jest.mock('../api/book', () => ({
  getBookList: jest.fn(() => {
    data: [{ name: '你不知道的JavaScript' }]
  }),
}))

test('should fetch book list', async () => {
  const payload = { category: '文学' }

  const { storeState } = await expectSaga(sagas).withReducer(reducer).dispatch({ type: types.FETCH, payload }).run()

  expect(service.getAllBooks).toBeCalledWith(payload)
  expect(storeState).toEqual({
    list: [{ name: '你不知道的JavaScript' }],
    total: 1,
    error: null,
  })
})
```

## selectors 测试

```js
// book.js
export const getBookList = (state) => state.book.list

export const getBooksByCategory = createSelector(
  getBookList,
  (_, category) => category,
  (books, category) => books.filter((book) => book.category === category),
)
```

Note: selector 的测试与 mutation 一样直截了当。selectors 也是比较重逻辑的地方，并且它也是一个纯函数，与 reducers 测试享受同样待遇：纯净的输入输出，简易的测试准备。下面来看一个稍微简单点的 getters 测试用例：

```js
// book.test.js
test('should get book by category', () => {
  const category = '前端'

  expect(getBooksByCategory(state, category)).toEqual([{ name: '你不知道的JavaScript' }])
})
```

# React 组件和 Redux store 的交互

前面我们讲完了 Redux 单元测试所需要的基本知识，而 React 组件需要从 Redux store 读取状态或者是发送 action 改变 store 状态的时候，又该如何测试他们之间的交互呢？接下来就来聊聊如何用 React Test Utils 测试 React 组件中的 Redux。

```js
export function BookList({ category }) {
  const location = useLocation()
  const { tag } = queryString.parse(location.search)

  const booksByCategory = useSelector((state) => getBooksByCategory(state, category))
  const books = tag ? booksByCategory.filter((book) => book.tags.includes(tag)) : booksByCategory

  return <>...</>
}
```

站在单元测试的角度，其实我们在测试 React 组件（单元）的时候不需要关心 Redux store 长什么样子，我们只需要知道 Redux store 当中的这些 action 将会在适当的时机触发，以及它们触发时的预期行为是什么。

## ReduxWrapper renderWithRedux()

```js
export function ReduxWrapper({ initialState, store = mockStore(initialState), children }) {
  return <Provider store={store}>{children}</Provider>
}

export function renderWithRedux(ui, { initialState, store = mockStore(initialState) } = {}) {
  return {
    ...render(
      <ReduxWrapper initialState={initialState} store={store}>
        {ui}
      </ReduxWrapper>,
    ),
    store,
  }
}
```

在单元测试的时候，shallowMount（浅渲染）方法接受一个挂载 options，可以用来给 React 组件传递一个伪造的 store。然后我们就可以使用 Jest 模拟一个 action 的行为再传给 store，而 actionClick 这个伪造函数能够让我们去断言该 action 是否被调用过。所以我们在测试 action 的时候就可以只关心 action 的触发，而至于触发之后对 store 做了什么事情我们就不需要再关心了，因为 Redux 的单元测试会涵盖相关的代码逻辑。

## RouterWrapper renderWithRouter()

```js
export const RouterWrapper = ({
  route = '/',
  history = createMemoryHistory({ initialEntries: [route] }),
  children,
}) => <Router history={history}>{children}</Router>

export function renderWithRouter(ui, { route = '/', history = createMemoryHistory({ initialEntries: [route] }) } = {}) {
  return {
    ...render(
      <RouterWrapper route={route} history={history}>
        {ui}
      </RouterWrapper>,
    ),
    history,
  }
}
```

# 总结一下

- **# 单元测试基础**
- **# React 单元测试**
- **# Redux 单元测试**
- **# React 应用测试策略**

## 🤓 总结：快速响应变化，缩短反馈周期

- 缺少**可重构性**的软件，不可能快速响应变化。
- 没有高覆盖率、快速运行的**单元测试**，重构就不可能落地。
- **测试驱动开发**是获得高质量单元测试集的唯一有效方法。
- 建立在充分覆盖且运行快速的自动化测试基础上的**持续集成**是迭代式开发的必要条件。

> 缺陷发现越早，修复成本越低

## 附录：武林秘籍 📓

### Tasking 任务分解

- 完全穷尽
- 相互独立

### Testing 单元测试

- Given
- When
- Then

### Refactoring 重构

- 旧的不变，新的创建；
- 一步切换，旧的再见。

![img](https://jimmylv.gitee.io/slides/images/tdd/Test-Driven-Development-Cycle.jpg)

# 📚 推荐书籍

- [React 官方中文文档 – 用于构建用户界面的 JavaScript 库](https://zh-hans.reactjs.org/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- 《深入 React 技术栈》
- 《深入浅出 React 和 Redux》
- 《重构 2：改善既有代码的设计》
- 《Clean Code 代码整洁之道》
- 《Refactoring to Patterns 重构与模式》
- 《SICP 计算机程序的构造和解释》
