# 拖拽自定义报表

## 介绍

该项目起源于公司内部的需求，本人在司维护开发的项目具有大量相似的报表页面，而最近又接到三个雷同页面，该项目应运而生。

项目在电脑端，遂工具制作成了拖动完成排版的形式，方便可爱的运维小姐姐操作。

推荐使用 `chrome` 浏览器食用更佳。

## 演示

gif 图稍大，若加载不出来请稍等片刻 (..•˘_˘•..)

![演示1](https://i.loli.net/2019/03/12/5c8719e48a970.gif)

![演示2](https://i.loli.net/2019/03/12/5c8719e30304c.gif)

## 说明

在拖动页面实际拖动的就是一个拥有背景的 `div`，在预览页面才会生成其对应的组件。

项目主要由两个页面构成，第一个页面主要为拖动编辑排版，第二个页面为预览编辑的页面。

### 插件说明

- `echarts` 百度可视化图表库
- `element-ui` 饿了么组件库
- `ramda` 函数式编程顶好的库之一
- `jspdf` 结合 `html2canvas` 实现 pdf 导出功能
- `html2canvas` 结合 `jspdf` 实现 pdf 导出功能
- `lodash` 暂且只使用了 debounce 函数
- `mockjs` 前端随机数据生成工具

### 结构说明

#### 拖动页面

通识

- 页面由行（`row`）组成，每一行又由列又称组件（`col`）组成
- 将 `row` 栅格化，可以理解成 `24` 个块，组合可以随意，`12 + 12`、`16 + 8` 等，但其实 `8 + 8` 也可，行内排序改为两侧留白或者两侧对齐即可
- `row` 间可以进行拖动更改位置，`col` 间可以进行拖动更改位置
- `v-if` 和 `v-show` 的区别，在于是否会销毁某标签，使用 `v-show` 可以保存组件更改

使用的 `API` 说明

- `draggable: true` 标识该标签内容是否可被拖动
- `dragstart` 某一可拖动组件被拖动时，该事件会触发
- `drop` 当某一可拖动组件被拖动至某一标签上时，该事件会被触发
- `dragover` 由于拖动需要阻止事件的默认事件，遂一般会配合 `drop` 食用

行 --- 添加、修改、删除

- 添加一行，点击左侧的 `+`，拖动某一行排序至中间的行即可
- 修改某一行，鼠标放置某一行上，顶部浮现出行-控制条，拖动该控制条至其他行即可交换两行位置
- 删除某一行，鼠标放置某一行，顶部浮现出行-控制条，点击 `remove` 即可

列 --- 添加、修改、删除

- 添加一组件，点击右侧的 `+`，拖动某一组件至中间的位置即可
- 修改某一组件，鼠标直接拖动某一组件至想要的位置即可
- 删除某一组件，鼠标放置某一组件上，顶部浮现出列-控制条，点击 `remove` 即可

#### 预览页面

通识

- Vue 动态组件

使用的 `API` 说明

- `<component is='component-title'/>` [`Vue` 动态组件](https://cn.vuejs.org/v2/guide/components.html#%E5%8A%A8%E6%80%81%E7%BB%84%E4%BB%B6)

组件集（`src\components\custom-report`）

- 带 `custom-report-` 前缀的 `vue` 文件都会被自动引入到预览页面成为一个可供使用的组件
- 带 `default-` 的组件需要手动引入
  - 基础 `echarts` 组件，在同级目录下的 `mixins` 中有定义其需要混入的方法
  - 基础 `element-table` 组件，接受一个 `table-option` 来定义表格的表头和 `key` 内容，由于该类型页面多为展示，遂该组件没有监听事件，若需要监听多个 `element-table` 的事件也简单，`element-table` 加上 `v-on="$listeners"` 即可
- `echarts` 组件简单说明
  - `echarts` 在 `src\prototype` 中被挂载到 `Vue` 的实例原型上方便调用
  - 为什么会有这么多次的 `setOption`
    - 第一次 `defaultOption` 设置默认 `option`（在该目录下的 `js\chart-variable`），同时想要修改图表颜色集也可以在这里修改
    - 第二次 `customOption` 某些页面的图表使用默认的 `option` 会导致样式错位，这时候就需要增加 `js\variable` 变量，用于设置一些特殊的排版，比如 `center: [50%, 50%]`，等
    - 第三次 `reportData`，这个时候才是真实数据应该渲染的时候，建议在此处不要修改 `echarts` 样式相关的内容，而是只注入 `data`

### 数据格式说明

这里会详细说一下数据格式，具体可以参阅（`src\mock\modules\variable`）文件。

#### 行数据

```js
{
    // 表格 title
    title: 'drag-report',
    // 表格标识 key 通常唯一
    reportKey: 'first-report',
    // 行数据
    children: [{
        // 行的水平排序方式
        align: 'flex-start',
        // 行高，暂时只有 250 和 100 两种
        height: 250,
        // 行 index 用于排序
        index: 1,
        // 行内组件数据
        children: [{
            // 组件标题
            title: '评分图',
            // 组件行占比
            col: 24,
            // 组件 key 用于查询组件
            componentKey: 15,
            // 推荐行占比
            initCol: 24
        }]
    }]
}
```

#### 组件数据

```js
{
    // 组建默认名
    label: '占位',
    // 组件 key 通常唯一
    componentKey: 1,
    // 该组件对应的 vue 组件名称，在动态组件渲染的时候会自动加上 custom-report- 前缀
    componentName: 'block-module',
    // 该组件对应的接口，接口相同也没事，相同的请求不会发起两次
    api: '',
    // 该接口的类型
    method: 'get',
    // 返回的数据中，需要的那个数据的 key，若为空组件内会直接接受全量数据
    // 比如返回的 responseData 为 { names: [{name: 'king'}, {name: 'kimi'}] }
    // dataKey 就可以设置为 name，所对应的 vue 组件的 prop reportData 就会接受到 name 的数据
    dataKey: '',
    // 组件的行占比
    col: 1,
    // 组件的高度，暂时没用，可用于筛选
    height: 250,
    // 组件的预览图，用于拖动页面显示
    previewImage: 'https://i.loli.net/2019/03/11/5c8663be38f28.png'
}
```

### 小部件说明

`clickoutside` 来自于 `element-ui` 的源码，用于判断用户是否点击某一元素。

### 优化说明

本来的想法是一个组件对应一个接口，现在对相同接口的组件进行了优化，如果接口相同则只会调用一遍接口。

## 代办事项

- 调用接口所传输的数据需要相同，拟改为可以自定义传输的数据
- 拟把行的高度也改为可以自定义的，这时候就需要一个行内垂直的排序方式
- `pdf` 导出的样式有点丑，后面稍微改一下
- 拟将 `drag` 改为 `mousedown` 等，进行低版本浏览器的兼容
- 拟增加一个预览页面切换报表颜色系的功能

## 后语

- 该项目中困难的并非是该如何实现拖拽功能，真正困难的是数据格式的设计，由于该工具为前端主导后端为辅，数据格式大的调整有过三次，一次是格式问题，不能实现我想要的功能，两次是格式优化，考虑到要存储到数据库，遂对所需格式又进行了两次优化
- 考虑到数据应该都由后台接口获取过来，所以加入了 `mockjs` 用以模拟 `ajax` 请求，引入到正式项目中的时候可以减少更改
- 样式整理也是一个比较令人头秃的一个点，在初版完成之后的改版优化中，我一边改一边骂自己是个傻 X，还好有 `sass` 的存在，让我减少了不少工作量，但是不能否认的是项目中还是有许多辣鸡代码（何谓辣鸡代码，就是让我看一遍也会想拍死作者，即使是自己写的），还是需要做优化
- 在预览报表的页面，由于了解过 `Vue` 的 `component` 组件，所以实现起来也不是什么难事，关键点在于每个组件对外接收的 `prop` 要做到统一，这不仅是为了方便，任何一个框架都需要有人来维护，这也是为了减少后面接手的人的理解成本
- 在框架搭建完成之后，组件式报表的业务重点就在于大量的组件，本来的写法是写一个组件在预览页面引用一个，但是在看 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin) 的源码的时候，发现了 `webpack` 的 `require.context` 这个令人着迷的 `API`，那还能忍？盘他，于是，就有了 `src\views\previewReport\index.js` 这个文件的诞生，只要组件命名规范（前缀 `custom-report-`），该文件就会自动引入该组件，方便省心，比充 X 娃娃还好使
- 由于这样的页面大都不仅是用来看看，都会结合一下 `pdf` 导出的功能，遂查阅相关资料做了一个 `pdf` 导出的功能，前端实现 `pdf` 导出功能的思想就是通过遍历 `dom` 来生成 `canvas`，然后又借助于 `canvas` 的 `API` 获取图片的 `base64` 码，最后通过 `jspdf` 生成 `pdf`