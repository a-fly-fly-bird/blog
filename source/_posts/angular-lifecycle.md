---
title: Angular 生命周期
toc: true
cover: https://source.unsplash.com/random
tags: ['Angular']
categories: ['前端']
date: 2024-05-28 23:41:48
---

Angular新手可能经常会有和我一样的疑问，我明明已经设置了值呀，为什么还是报读到`undefined`错误呢？有两种常见的原因，异步或者生命周期。今天就先来深入了解一下Angular的生命周期（适用于组件，指令等，因为本质上来说，组件也是指令）。

<!-- more -->

# Angular Lifecycle

官方文档：[组件的生命周期](https://angular.cn/guide/lifecycle-hooks)。

| 钩子方法                | 用途                                                                                                                                                                                                             | 时机                                                                                                                                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ngOnChanges()           | 当 Angular 设置或重新设置数据绑定的输入属性时响应。该方法接受当前和上一属性值的 SimpleChanges对象 注意： 这发生得比较频繁，所以你在这里执行的任何操作都会显著影响性能。 欲知详情，参阅本文档的使用变更检测钩子。 | 如果组件绑定过输入属性，那么在 ngOnInit() 之前以及所绑定的一个或多个输入属性的值发生变化时都会调用。 注意： 如果你的组件没有输入属性，或者你使用它时没有提供任何输入属性，那么框架就不会调用ngOnChanges()。 |
| ngOnInit()              | 在 Angular 第一次显示数据绑定和设置指令/组件的输入属性之后，初始化指令/组件。欲知详情，参阅本文档中的初始化组件或指令。                                                                                          | 在第一轮 ngOnChanges() 完成之后调用，只调用一次。而且即使没有调用过 ngOnChanges()，也仍然会调用 ngOnInit()（比如当模板中没有绑定任何输入属性时）。                                                          |
| ngDoCheck()             | 检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应。欲知详情和范例，参阅本文档中的自定义变更检测。                                                                                                      | 紧跟在每次执行变更检测时的 ngOnChanges() 和 首次执行变更检测时的 ngOnInit() 后调用。                                                                                                                        |
| ngAfterContentInit()    | 当 Angular 把外部内容投影进组件视图或指令所在的视图之后调用。 欲知详情和范例，参阅本文档中的响应内容中的变更。                                                                                                   | 第一次 ngDoCheck() 之后调用，只调用一次。                                                                                                                                                                   |
| ngAfterContentChecked() | 每当 Angular 检查完被投影到组件或指令中的内容之后调用。 欲知详情和范例，参阅本文档中的响应被投影内容的变更。                                                                                                     | ngAfterContentInit() 和每次 ngDoCheck() 之后调用。                                                                                                                                                          |
| ngAfterViewInit()       | 当 Angular 初始化完组件视图及其子视图或包含该指令的视图之后调用。 欲知详情和范例，参阅本文档中的响应视图变更。                                                                                                   | 第一次 ngAfterContentChecked() 之后调用，只调用一次。                                                                                                                                                       |
| ngAfterViewChecked()    | 每当 Angular 做完组件视图和子视图或包含该指令的视图的变更检测之后调用。                                                                                                                                          | ngAfterViewInit() 和每次 ngAfterContentChecked() 之后调用。                                                                                                                                                 |
| ngOnDestroy()           | 每当 Angular 每次销毁指令/组件之前调用并清扫。在这儿反订阅可观察对象和分离事件处理器，以防内存泄漏。欲知详情，参阅本文档中的在实例销毁时进行清理。                                                               | 在 Angular 销毁指令或组件之前立即调用。                                                                                                                                                                     |

## OnChanges
注意，只有在**当前组件或者指令的@Input/@Output绑定的值变化时**会触发这个函数。另外需要注意的是，如果 @Input 是个对象，对象里面的数据改变但是引用没有变化也不会触发这个函数(这就是为什么经常需要使用[lodash的`cloneDeep()`](https://www.lodashjs.com/docs/lodash.cloneDeep)函数进行**深拷贝**的原因，本质是改变引用地址)。

## DoCheck
[Angular：ngDoCheck执行时机](https://limeii.github.io/2019/06/angular-ngdocheck-onpush-strategy/)

# 父子组件生命周期

参考自：
- [Angular--父子组件生命周期钩子(lifecycle hooks)执行过程](https://www.cnblogs.com/sparkler/p/16864656.html)
- [Angular Lifecycle Execution flow from parent to child component](https://chauhansawatantra.medium.com/angular-lifecycle-execution-flow-from-parent-to-child-component-f6303c42478)

![parent child lifecycle](image.png)

