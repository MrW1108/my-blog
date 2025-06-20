---
title: Elpis 学习心得
top: true
cover: true
date: 2025-6-19
summary: 基于 node + vue3 + webpack5 ，完成一个名为 elpis 的全栈开发框架。Elpis 思想：通过 80% 的抽离和重复工作支持配置化，20% 的定制化工作，减少 CRUD 体力活的比例，让程序员更专注于创造
categories: 前端
tags:
  - 实践
  - node
  - koa
  - vue3
  - webpack5
---

# Elpis 学习心得

## 一、背景

### 1.1 当前大多数前端开发的痛点

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/1-elpis.png) 随着前端框架的成熟，开发者们越来越多地陷入了重复的 CRUD 工作中，忽略了程序员的本质：7 分构想，3 分代码。Elpis 的出现，旨在分享一种思想：通过 80% 的抽离和重复工作支持配置化，20% 的定制化工作，减少 CRUD 体力活的比例，让程序员更专注于创造。

### 1.2 庞大且功能全的系统

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/2-elpis.png) 缺点：

* 项目体积大
* 不适合**多用户**交付场景
* 交付时候，可能会包含**无用的代码**
* 定制化能力弱，往往**需要用户妥协**
* 系统迭代 **熵增明显**

### 1.3 多子系统：类似微前端

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/3-elpis.png) 缺点：

* 通用的建站能力不适合**领域强的场景**
* 过分灵活，配置复杂，**若处理不好容易代码断代，此时对提效没有实质的提高，反而成了负担**
* 不能体系化触达到领域问题

### 1.4 elpis思想

融合前面所提到的庞大系统（不灵活），多个子系统（过分灵活）的特点，用折中的方案去优化开发面临的问题，

* 粒子：算子服务
* AOP领域模型
* 面向对象建站

#### AOP领域模型

* **业务为中心**
    
    * **角色升级：从技术执行者到业务设计者**  
        程序员不再局限于编写代码或被动理解需求，而是需要基于对行业的深度认知，主动抽象业务本质，构建高内聚、低耦合的领域模型，驱动技术方案与业务目标对齐。
    * **模型复用：沉淀行业解决方案**  
        通过提炼共性业务规则与流程，形成可复用的基础领域模型，后续针对不同系统只需在基础模型上扩展差异化逻辑，而非从零开发，提升交付效率与系统一致性。
* **高内聚、低耦合**
    
    * 将同一业务领域的操作逻辑与数据高度聚合，通过标准化接口（如 JSON Schema）暴露能力，降低模块间依赖。
    * **业务能力封装**：将数据校验、状态流转、业务规则等逻辑内聚在模型中，对外提供声明式接口。
    * **依赖隔离**：模型内部处理业务逻辑（如库存扣减），外部系统通过 Schema 调用接口，无需感知实现细节。
* **明确边界**
    
    * 按业务语义划分模型边界，避免功能混杂，确保单一模型仅解决特定领域问题。
    * 例如电商行业的商品管理、订单管理和客户管理是每个电商系统都会用到的，课程管理系统的视频管理、用户管理则可以分别弄成一个基础的模型
* **领域模型的工程价值**
    
    * **降低认知成本**：业务逻辑集中管理，新成员通过模型代码快速理解业务规则。
        
    * **提升交付效率**：复用基础模型减少重复开发，例如新建教育系统时直接复用用户模型。
        
    * **增强系统弹性**：模型边界清晰，单一业务变更（如修改支付流程）不会波及订单履约模块。
        

#### 面向对象建站

* 创新采用配置驱动开发模式，通过动态渲染引擎实现JSON配置到xxx项目（不同领域有不同的**解析器**）进行自动解析，需求交付周期缩短60%

## 二、内核引擎应用开发

### 2.1 宏观角度

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/4-elpis.png)

* **通过约定的规则**，在对应的文件目录放置对应的模块
* 通过elpis-core 对文件进行解析挂载到app对象中
* 在进行编码时候通过app就可以获取到我们想要使用的能力

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/5-elpis.png)

#### 1.接入层

* 接口路由处理，区分API请求，页面请求
* 路由规则校验：请求是否满足最基本的校验
* 路由中间件：是否携带所需的参数，认证是否通过等

#### 2.业务层

* controller 处理器
* env 环境区分
* config 全局变量提取
* extend 服务拓展插件
* schedule 定时任务

#### 3.服务层

* service处理器，是跟数据层的交互

### 2.2 解析器

**Elpis的主要功能之一就是实现下面的解析器**

* router-loader 解析router请求文件
    * 页面请求，
    * API请求
* router-schema-loader:解析API参数类型校验文件
* middleware-loader: 解析自定义中间件文件
* controller-loader: 解析逻辑处理文件
* service-loader: 解析服务文件（日志，mysql,调用外部服务等）
* extend-loader: 解析扩展文件
* config-loader: 解析配置文件

### 2.3 koa洋葱模型

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/6-elpis.png)

* 业务逻辑前遵从：**先用先处理**
* 业务逻辑后遵从：**后用先处理**

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/7-elpis.png)

## 三、工程化

### 3.1 工程化的核心

* 我们编写的代码让浏览器（或者说某个应用）能够识别，工程化就是充当这样的一个中介
* 工作提效，一步到位通过一条命令，执行多个配置好的操作，直接打包出产物
* 对多种写法进行兼容
* 某个框架的盛行，社区的庞大，无形中形成开发的规范
* 看完这章节的讲解：用哪个框架不重要，如何处理的思想是在任何一个框架中都想通的，**哪个框架你用起来顺手能解决你的问题都可以** 无需纠结哪个框架，思想最重要

### 3.2 流程

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/8-elpis.png)

**其中模版解析就是工程化所需要处理的**

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/9-elpis.png)

### 3.3 热更新

**热更新一般是在开发时候使用**

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/10-elpis.png)

**原理**

首先通过搭设一台服务器devServer 通过该服务器作为中转站进行通知发送和监控业务文件

* 1.监控业务文件变化
* 2.业务文件变化后通知打包引擎进行打包，产物放入内存
* 3.通知内存文件更新
* 4.产物文件进行重新注入

**实现** 本项目是通过express框架实现该功能，而不是使用webpack5自带的devServer配置

* 1.使用webpack-dev-middleware作为监听业务文件变化的中间件
* 2.webpack-hot-middleware用于通知内存文件更新的信息
* 3.webpack.HotModuleReplacementPlugin 用于实现热模块替换(Hot Module Replacement 简称 HMR)基于模块的角度去更新，不会更新整个模块

### 3.4 优化

这项目虽然使用的是webpack5，但是不仅仅局限这个工具，**只要能实现你要的效果都可以**

工程化优化从2个方面思考

#### 1\. 构建速度优化

* 1.devtool:映射源的选择 不同环境使用不同的策略，开发环境可以全面一点，生产环境可以不映射，甚至不让使用者看到source
* 2.使用构建缓存 Cache
* 3.loader细分需要构建的范围
    * include exclude
    * oneOf范围锁定
* 4.多线程处理loader
    * thread-loader
    * happyPack
* 5.压缩资源的插件尽量使用多线程

> 问题：使用多线程打包一定是更快吗？

否：首先线程的启动需要时间，若打包很小的模块，反而是负担，时间会更长。 所以一般建议在一些耗时的打包过程开启多线程，另外还可以**通过对比loader plugin开启多线程后的打包时间对比决定了是否开启**

#### 2.首屏渲染加载优化

* 1.合理使用浏览器缓存机制：注意打包文件时候contenthash,hash,chunHash的设置
* 2.静态资源的压缩（建议在打包到生产才开启）
    * js：terser
    * css:css-minimizer-webpack-plugin
    * image:image-minimizer-webpack-plugin等插件只是辅助作用

> 图片压缩一定在webpack打包完成吗？

个人觉得image-minimizer-webpack-plugin这些压缩挺费时间，静态资源图片若可以，我建议使用tinypng压缩即可，之前我工作中ui已经帮我压缩的很极致。

* 3.tree-shaking（树摇）运用
* 4.分包：目的使用浏览器可以并行下载的优势，vue中路由的懒加载也是这个原理
    * 第三方库为一个包
    * 业务代码为一个包
    * 公用方法一个包

> 这里注意若elementui使用的是按需加载，这时候建议单独分为一个包，因为会因为你加载的组件前后不一，第三方库的包contenthash会改变，会导致缓存失效

> 包分的越多越好？首先服务器不支持http2.0的情况不支持多路复用，浏览器（谷歌）只能同时并发同域名的6个请求，而且启动请求还需要tcp握手等操作，所以包并不是越多越好，还需要考虑包的大小

* 5.项目中合理使用preload和 prefetch加载资源

## 四、Dashboard模版

当前开发一个系统（以中后台系统为例子），大部分程序员都是产品说出一个页面的需求，程序员看到什么就开始开发什么，这样会导致大量的重复CRUD的工作，对自身的成长没有什么帮助 有点经验的，就会根据自身的开发习惯，对页面部分对页面进行封装，通过配置的方式渲染页面，虽然更灵活了减少了一定的工作量，但这种往往难以形成统一的风格，往往需要根据需求进行妥协，在开发到一定时间后，组件反而会很冗余不好维护

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/11-elpis.png)

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/12-elpis.png) 

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/13-elpis.png)

### 4.1 在接触elpis前，自己设计的大屏系统也有类似的想法，以下是大致的描述

通过布局组件（layout-comp）将页面分为24\*24的区域，利用动态组件+插槽的方式保证系统的灵活性

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/14-elpis.png)

通过配置的方式对区域大小进行渲染

* 左1描述

```js
chartsData: [
  {
    // 左1
    grid: {
      x: 0,    // 向左偏移0 个单元格
      y: 0,   // 向上偏移0 个单元格
      w: 12,   // 宽度占12个单元格
      h: 12,  // 高度占纵向的12个单元格
    },
    key:'0',
    linkKey:'', // 关联的key,用于实现数据联动操作影响到哪个组件
    bcType: null, // border边框类型, image背景图, null 不做操作
    bc:'', // 对应bcType类型
    children: [] // 内嵌块，布局组件也将内部块分为24*24个格子，
    // 将其封装在一个数组中，最后再进行渲染
  },
  ...
]


```

* 右1描述

```js
chartsData: [
   {
    // 右1
    grid: {
      x: 12,    // 向左偏移0 个单元格
      y: 0,   // 向上偏移0 个单元格
      w: 12,   // 宽度占12个单元格
      h: 12,  // 高度占纵向的12个单元格

    },
    key:'1',
    bcType: null, // border边框类型, image背景图, null 不做操作
    children: [
      {
        // 定位于 右1描述方框
        grid:{
          x: 0,    // 向左偏移0 个单元格
          y: 0,   // 向上偏移0 个单元格
          w: 24,   // 宽度占24个单元格
          h: 12,  // 高度占纵向的12个单元格
        },
        key:'1-1',
        linkKey:'', // 关联的key,用于实现数据联动操作影响到哪个组件
        compType:'柱状图', // 伪代码
        data:[], // 数据放在这参数上。这里省略只是描述出来有这个字段
        compStyle:{
          color:'xxxx',
          fontSize:'xxxx'
        }, // 伪代码
      },
      {
        // 定位于 右1描述方框
        grid:{
          x: 0,    // 向左偏移0 个单元格
          y: 12,   // 向上偏移12 个单元格
          w: 24,   // 宽度占24个单元格
          h: 12,  // 高度占纵向的12个单元格
        },
        key:'1-2',
        linkKey:'', // 关联的key,用于实现数据联动操作影响到哪个组件
        compType:'饼图组件', // 伪代码用来定义需要渲染的组件
        data:[], // 数据放在这参数上。这里省略只是描述出来有这个字段
        compStyle:{
          color:'xxxx',
          fontSize:'xxxx'
        }, // 伪代码
      }
    ]
  },
  ...
]

```

* 右下一描述

```js

chartsData: [
 {
    // 右下1
    grid: {
      x: 12,    // 向左偏移0 个单元格
      y: 12,   // 向上偏移0 个单元格
      w: 12,   // 宽度占12个单元格
      h: 12,  // 高度占纵向的12个单元格

    },
    key:'3',
    linkKey:'', // 关联的key,用于实现数据联动操作影响到哪个组件
    bcType: null, // border边框类型, image背景图, null 不做操作
    children: [
      {
        // 定位于 右下1描述方框
        grid:{
          x: 0,    // 向左偏移0 个单元格
          y: 0,   // 向上偏移0 个单元格
          w: 24,   // 宽度占24个单元格
          h: 6,  // 高度占纵向的6个单元格
        },
        key:'3-0',
        linkKey:'3-1', // 关联的key,用于实现数据联动操作影响到哪个组件
        compType:'统计文字', // 伪代码
        data:[], // 数据这里省略只是描述出来有这个字段
        compStyle:{
          color:'xxxx',
          fontSize:'xxxx'
        }, // 伪代码
      },
      {
        // 定位于 右下1描述方框
        grid:{
          x: 0,    // 向左偏移0 个单元格
          y: 6,   // 向上偏移6 个单元格
          w: 24,   // 宽度占24个单元格
          h: 18,  // 高度占纵向的18个单元格
        },
        key:'3-1',
        linkKey:'', // 关联的key,用于实现数据联动操作影响到哪个组件
        compType:'关联列表', // 伪代码
        data:[], // 数据这里省略只是描述出来有这个字段
        compStyle:{
          color:'xxxx',
          fontSize:'xxxx'
        }, // 伪代码
      }
    ]
  }
  ...
]

```

#### **遇到的问题**

##### 1\. **需求对齐与设计规范固化**

* **问题**：研发初期仅与UI单点对齐，缺乏产品侧对系统能力的全局输入，导致不同产品设计大屏时交互逻辑碎片化，DSL被迫承载过多兼容逻辑，维护成本攀升。
    
* **优化方案**：
    
    * **三方协作机制**：建立产品-研发-UI的定期需求同步会，明确系统当前支持的交互范式与技术边界。
        
    * **DSL分层设计**：
        
        * **核心层**：固化通用交互标准（如数据筛选、图表联动），尽可能减少业务定制。
        * **扩展层**：通过插件机制支持低频定制需求，避免污染核心逻辑。
    * **设计系统落地**：将高频交互场景沉淀为可视化配置项，产品直接基于标准模板设计，减少技术兜底。
        

##### **2\. 数据接口协作模式升级**

* **问题**：依赖文档传递数据规范，后端需反复理解适配，协作效率低下。
    
* **优化方案**：
    
    * **BFF层抽象**：在前端与后端间增加BFF（Backend for Frontend）层，实现以下能力：
        
        * **接口聚合**：将多个微服务接口合并为前端友好格式。
        * **数据转换**：统一驼峰/下划线命名、枚举值映射等差异。
        * **Mock机制**：基于契约自动生成模拟数据，前端不阻塞开发。
    * **协作分工**：
        
        * 后端：仅保障原始数据准确性（如SQL执行效率、事务一致性）。
        * BFF层：由前端主导开发，负责业务数据适配。

##### **3\. 组件分层架构优化**

* **问题**：业务逻辑侵入基础组件（如弹窗包含审批流），导致复用性与维护性下降。
    
* **优化方案**：
    
    * **三层架构设计**：
        
        | **层级** | **职责** | **示例** |
        | --- | --- | --- |
        | **基础组件层** | 纯UI渲染，无业务逻辑 | `<Modal>` 仅处理显示/隐藏动画 |
        | **业务桥接层** | 处理数据转换、事件分发 | `ApprovalModalAdapter` 对接审批接口 |
        | **场景封装层** | 组合多组件实现完整业务流 | `LeaveApprovalFlow` 包含表单、弹窗、状态机 |
        
    * **事件驱动通信**：组件通过自定义事件（如 `@submit`）与桥接层交互，彻底解耦逻辑与视图。
        
* 样式定义（compStyle）为了灵活，图表的参数遵从了echarts官网的参数,这导致了学习成本提高，因为在使用过程中发现，每个系统所需要的图表风格较为相似应该 做多一个样式解析器即可，降低配置人员的学习成本
    

##### **4\. 可视化配置体验优化**

* **问题**：直接暴露ECharts复杂参数，配置门槛高且重复劳动。
    
* **优化方案**：
    
    * **配置分层策略**：
        
        * **基础样式包**：封装通用主题（颜色、字体、间距），通过 `compStyle` 一键应用。
        * **语义化配置器**：将ECharts参数转换为业务术语（如「指标轴」替代 `yAxis.name`）
        * **模板中心**：沉淀各系统高频图表配置（如「销售漏斗」「用户留存曲线」），支持克隆后微调。

### 4.2 DSL

DSL是一种专门为某个特定领域或问题域设计的编程语言。它的主要目的是提供一种简洁、易于使用和高效的方式来描述和解决特定领域的问题。本框架通过dsl ->解析器->模版页面

大致框架如下：**其中绿色为自定义组件**：可自行变动保证灵活性，**蓝色部分为已设计部分**：固定部分减少重复操作

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/15-elpis.png)

#### 1\. elpis核心：80%配置减少重复工作， 20%定制化增加系统的拓展性

通过**json-schema**的定义规范对字段进行描述

上面是对管理后台系统为背景进行的DSL设计

##### 1.1菜单栏参数设计

```js

menu:[
  {
     key:'',// 每个菜单的key
     name:'',
     menuType:'', // module group 根据不同的值用到下面不同的数据
     subMenu:[{...}] // 可递归的menuItem
     moduleType:'',// 当menuType 为module时， 枚举值: sider / iframe / custom / schema
     siderConfig:{...} // 对应 sider
     iframeConfig:{...} // 对应iframe
     customConfig:{...} // 对应custom
     schemaConfig:{...} // 对应schema的时候
  }
]

```

##### 1.2页面布局组件

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/16-elpis.png)

* header-view用来处理菜单栏的点击逻辑，并根据所点击的键去发送事件出上层组件影响 slot:main-content渲染内容
* header-container 一个纯进行渲染的组件不进行逻辑处理
* slot:menu-content 根据 menu\[...\]进行菜单的渲染
* slot:setting-content 保留给用户自定义的站位
* slot:main-content : 根据moduleType的不同 main-content 会定向到不同的页面，其中**schema类型将是重点**

#### iframe-view

当 moduleType为 iframe 读取iframeConfg.path iframe进行渲染

#### custom-view

当 moduleType为 custom 读取customConfg.path 路由进行csr (用于定制页面)

#### schema-view：重点

当 moduleType为 schema时，我们就需要对schemaConfig参数进行解析

* api
    * 该页面涉及到的api,遵从Restful规范
* schema
    * 数据源参数的描述，可以理解为api返回的数据字段，我怎么展示
* tableConfig
    * 表单配置，头部按钮组件返回什么事件，row按钮会触发的事件等
* searchConfig
    * search-bar 板块相关配置
* components
    * 页面涉及到的子组件

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/17-elpis.png)

大致可以理解为这样一张图，核心就是我需要哪个模块，我就新增一个配置用来描述，通过解析器解析成对应的模块

我需要什么就新增一个描述，**通过解析器转换成渲染组件所需要的数据，然后进行渲染交互**

![](https://mrpaly.oss-cn-beijing.aliyuncs.com/Img/blogImg/18-elpis.png)

## 五.动态组件

### 5.1设计组件的考虑

* 考虑组件需要什么参数
    * 首先先要从组件功能进行考虑先进行一个大概的梳理
    * 禁止那种需要什么参数再进行补充的思路
* 考虑需要暴露什么事件
    * 考虑组件功能，会给外部带来怎么的联动功能
* 考虑暴露什么属性
    * 需要考虑在使用过程中需要用到内部的什么属性？切记返回对应的方法，禁止直接修改属性
* 考虑slot可拓展的东西
    * 这里就需要考虑什么部分是可以沉淀的通用配置，剩下的部分是灵活配置

### 5.2设计结构

```Javascript
{
  model: "xxx", //模版类型
  name: "", // 模版名称
  desc: "", // 模版描述
  icon: "", // 模版图标
  homePage: "", // 首页(项目配置)

  // 头部菜单 只展示moduleType:'schema'时候组件对应的参数
  menu: [
    {
      menuType: "module", // 菜单类型 枚举值 group / module
      moduleType: "schema", // 当 menuType 为 module 时,可填
      schemaConfig: {
        api: "xxxx",
        schema: {
          type: "object",
          properties: {
            // 对应的一个字段
            key: {
              // !!!! 这里是重点 命名规则： xxxOption
              // xxx:对应每个组件的名字
              // 字段在不同动态 component中的相关配置，前缀对应 componentConfig中的键值
              // 如：componentConfig.createForm,这里对应 createFormOption
              // 字段在createForm中的相关配置
              createFormOption: {
                ...eleComponentConfig, // 标准 el-component 配置
                comType: "", //控件类型 配置组件类型 input/select/....
                visible: true, // 是否显示 默认为 true 显示 （false ，表示不在表单中显示）
                disabled: false, // 是否禁用(true/false)  默认为 false
                default: "", // 默认值

                // compType === 'select' 时生效
                enumList: [], // 选项列表
              },
            },
            ...
          },
        },
        // 动态组件 相关配置,
        // 任何页面用到的动态组件都需要在这里配置
        componentConfig: {
          // 新增 create-form 表单相关配置
          createForm: {
            title: "表达配置", // 表单标题
            saveBtnText: "保存", // 保存按钮文本
          },
          // edit-form 表单相关配置
          editForm: {
            mainKey: "", // 表单主键,用于唯一标识要修改的数据对象
            title: "", // 表单标题
            saveBtnText: "", // 保存按钮文本
          },
          // detail-panel 相关配置
          detailPanel: {
            mainKey: "", // 表单主键,用于唯一标识要修改的数据对象
            title: "", // 表单标题
          },
          // 支持动态配置
        },
      },
    },
  ],
};


```

### 5.3组件结构管理

```css
 app                                                                                     
│  │  ├─ pages                                                                        
│  │  │  ├─ dashboard                                               
│  │  │  │  ├─ complex-view                                                             
│  │  │  │  │  ├─ schema-view                                       
│  │  │  │  │  │  ├─ complex-view                                   
│  │  │  │  │  │  │  ├─ components                                  
│  │  │  │  │  │  │  │  ├─ create-form                              
│  │  │  │  │  │  │  │  │  └─ create-form.vue                       
│  │  │  │  │  │  │  │  ├─ detail-panel                             
│  │  │  │  │  │  │  │  │  └─ detail-panel.vue                      
│  │  │  │  │  │  │  │  ├─ edit-form                                
│  │  │  │  │  │  │  │  │  └─ edit.form.vue                         
│  │  │  │  │  │  │  │  └─ component-config.ts    
│  │  │  ├─ widgets                                                                    
│  │  │  │  ├─ schema-form                                          
│  │  │  │  │  ├─ complex-view                                      
│  │  │  │  │  │  ├─ input                                          
│  │  │  │  │  │  │  └─ input.vue                                   
│  │  │  │  │  │  ├─ input-number                                   
│  │  │  │  │  │  │  └─ input-number.vue                            
│  │  │  │  │  │  └─ select                                         
│  │  │  │  │  │     └─ select.vue                                  
│  │  │  │  │  ├─ form-item-config.ts                               
│  │  │  │  │  └─ schema-form.vue      

```

### 5.4为什么不用element-ui的表单校验功能

* 因为我们使用Json-schema的标准，Json-schema已经有很完善的验证机制（更好的做到数据驱动视图）
* 只需要结合ajv插件即可判断组件内部的数据是否符合我们的要求
* 另一层面可以统一团队的代码校验统一性，减少不必要的学习成本

## 六.npm封装打包

### 6.1npm封装打包目的

* 将一些重复性代码与业务需求的代码进行分离
* 利于项目的管理，只让elpis维护的成员能对沉淀的代码进行修改，使用者只需关心自身20%的业务逻辑

### 6.2npm封装，elpis需要考虑哪些方面

#### 6.2.1 node端

* 对外部文件的兼容 （支持业务controller、extend、router、router-schema、全局中间件、config配置）的拓展
* elpis-core内部路径指向修改，业务文件指向问题 （）， __dirname 和 process.cwd()区分

#### 6.2.2 webpack端

* 对之前的构建函数进行抽离暴露，业务端根据环境变量进行判断方法调用
    
* elpis的entry页面 和业务的entry页面抽离
    
* elpis方法和组件进行暴露
    
* babel-loader等插件使用require.resolve('xxx')进行引入
    
* 暴露自定义页面(外部定义路由，暴露方法)到entry.dashboard进行合并
    
* 暴露组件
    
    ```Javascript
       // 业务拓展 component 配置
       import BusinessComponentConfig from '$businessComponentConfig'
    
       const ComponentConfig:componentConfigType = {
         createForm:{
           component:createForm,
         },
         editForm:{
           component:editForm,
         },
         detailPanel:{
           component:detailPanel
         }
       }
       export default {
         ...ComponentConfig,
         ...BusinessComponentConfig
       }
       ```
    
    
    ```
    

### 6.3npm发布流程

* 注册一个npm账号
    
* Elpis 项目名称带上账号名称 @xxxx/elpis
    
* 本地调试
    
    * Elpis项目中执行npm link
    * Elpis-demo 项目 npm link @xxxx/elpis
* 调试完后 npm config get registry查看npm链接到哪个源
    
* npm config set registry **恢复默认源**
    
* npm login **登录npm**
    
* npm whoami **查看当前用户名**
    
* npm publish \*\*\*\***\--access public 首次发布需要加上后面这个后缀**
    

### 6.4达成elpis核心思想沉淀80% 20%灵活配置

* 1.通过npm打包将重复性的操作进行80%沉淀（每个公司都有自己的规则）业务性
* 2.每个项目通过引用 elpis包，然后自身的业务项目代码就通过elpis规则进行定义即可