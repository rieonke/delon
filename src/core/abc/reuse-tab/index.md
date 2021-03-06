---
title: reuse-tab
subtitle: 路由复用标签
order: 30
cols: 1
module: AdReuseTabModule
---

复用标签在中台系统非常常见，本质是解决不同路由页切换时组件数据不丢失问题。

> 控制台会打印很多日志，在组件未发布正式版之前，若发现BUG可以提供更多信息；`0.4.0` 正式版后将会移除。

## 匹配模式

在项目的任何位置（建议：`startup.service.ts`）注入 `ReuseTabService` 类，并设置 `mode` 属性，其值包括：

**1、（推荐，默认值）Menu**

按菜单 `Menu` 配置。

可复用：

```
{ text:'Dashboard' }
{ text:'Dashboard', reuse: true }
```

不可复用：

```
{ text:'Dashboard', reuse: false }
```

**2、（推荐）MenuForce**

按菜单 `Menu` 强制配置。

可复用：

```
{ text:'Dashboard', reuse: true }
```

不可复用：

```
{ text:'Dashboard' }
{ text:'Dashboard', reuse: false }
```

**3、URL**

对所有路由有效，可以配合 `excludes` 过滤无须复用路由。

> 除以上规则以外，路由配置中 `data` 属性若设置 `reuse` 值时优先级高于上述规则。

## 标签文本

根据以下顺序获取标签文本：

1. 组件内使用 `ReuseTabService.title = 'new title'` 重新指定文本，
2. 路由配置中 `data` 属性中包含 `reuseTitle` > `title`
3. 菜单数据中 `text` 属性

`ReuseTabService` 代码示例：

```ts
export class DemoReuseTabEditComponent implements OnInit {
    id: any;

    constructor(private route: ActivatedRoute, private reuseTabService: ReuseTabService) {}

    ngOnInit(): void {
        this.route.params.subscribe(params => {
            this.id = params.id;
            this.reuseTabService.title = `编辑 ${this.id}`;
        });
    }
}
```

## 常见问题

路由复用会保留组件状态，这可能会带来另一个弊端；复用过程中组件的生命周期勾子不会重复触发，大部分情况下都能正常运行，但可能需要注意：

- `OnDestroy` 可能会处理一些组件外部（例如：`body`）的样式等

而在组件内部唯一能够知道是否由复用产生的组件激活状态，只能透过 `Router.events` 事件监听；有兴趣可以扩展阅读 [ngx-ueditor](https://github.com/cipchk/ngx-ueditor/blob/master/lib/src/ueditor.component.ts) 的解决办法。

## API

### reuse-tab

参数 | 说明 | 类型 | 默认值
----|------|-----|------
max | 允许最多复用多少个页面 | `number` | `10`
excludes | 排除规则，限 `mode=URL` | `RegExp[]` | -
allowClose | 允许关闭 | `boolean` | `true`
showCurrent | 总是显示当前页 | `boolean` | `true`
fixed | 是否固定 | `boolean` | `true`
change | 切换时回调 | `EventEmitter` | -
close | 关闭回调 | `EventEmitter` | -

## 如何删除？

ng-alain 脚手架默认使用了 `reuse-tab` 组件，移除包含：

- `shared.module.ts` 的 `AdReuseTabModule` 模块导入语句及 `providers` 定义
- `layout.component.html` 的 `<reuse-tab></reuse-tab>` 组件模板定义
