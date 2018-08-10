react 生态圈的开放、自由、繁荣，也导致开发配置繁琐、选择迷茫。react-coat 放弃某些灵活性、以`约定替代某些配置`，固化某些`最佳实践`方案，从而提供给开发者一个更简洁的糖衣外套。

## 3.0.0 发布：

- 基于 2.0 的基本概念，进一步简化和清晰 API 的定义

## 3.1.0 发布：

- 对 ModuleAction 增加权限控制，对外暴露的 action 请使用 public 权限，model 内部调用的私有 action 请使用 protected 或 private 权限
- model 内部触发自已的 action，不再使用 this.actions.xxx()，改为：this.callThisAction(this.xxx)

```JS
 //3.0.0
yield this.put(this.actions.STARTED(this.state));

 //3.1.0
 yield this.put(this.callThisAction(this.STARTED, this.state));
```

## react-coat 特点：

- 集成 react、redux、redux-saga、react-router、history 等相关框架
- 仅为以上框架的糖衣外套，不改变其基本概念，无强侵入与破坏性
- 精简而自然的 API 语法，几乎不用学习即可上手
- 微框架，源码不到千行，编译成 ES5 并压缩后仅 15k 左右
- 业务模块化，支持按需加载
- 使用 typescript 强类型，更好的静态检查与智能提示

> 感谢 [Dva](https://github.com/dvajs/dva)带来的灵感，本框架与 Dva 主要差异：

- 更优雅和自然的 API
- 更清晰和简单的组织结构
- 使用 typescript 强类型推断和检查
- 路由支持按需加载，也支持整体加载
- 更大的灵活性和自由度，不强封装

```
// 比如：Dva中常这样写
dispatch({ type: 'moduleA/query', payload:{username:"jimmy"}} })

//本框架中可直接利用ts类型反射和检查:
dispatch(moduleA.actions.query({username:"jimmy"}))
```

## 安装 react-coat：

    $ yarn add react-coat

兼容 react 周边生态版本：

```
"peerDependencies": {
    "@types/history": "^4.0.0",
    "@types/react": "^16.0.0",
    "@types/react-dom": "^16.0.0",
    "@types/react-redux": "^5.0.0",
    "@types/react-router-dom": "^4.0.0",
    "connected-react-router": "^4.0.0",
    "history": "^4.0.0",
    "react": "^16.0.0",
    "react-dom": "^16.0.0",
    "react-redux": "^5.0.0",
    "react-router-dom": "^4.0.0",
    "redux": "^3.0.0 || ^4.0.0",
    "redux-saga": "^0.16.0"
  },
```

#### 套装版 react-coat-pkg

如果你想省心，你也可以直接安装"all in 1"的 [react-coat-pkg](https://github.com/wooline/react-coat-pkg)，它将自动包含以上组件，并保证各组件版本不会冲突：

    $ yarn add react-coat-pkg

### 兼容性：

IE9 或 IE9 以上

本框架依赖于浏览器 API "Promise"，低版本浏览器请自行安装 polyfill

# 使用 react-coat：

> 快速上手：[一个简单的 Hello Word](https://github.com/wooline/react-coat-demo-simple)

> 进阶：[使用 react-coat 重构 antd-pro](https://github.com/wooline/react-coat-antd)

> 为何需要此框架？参考：[支付宝前端应用架构的发展和选择](https://github.com/sorrycc/blog/issues/6)

### Module 概念

> react-coat 建议将复杂的业务场景分解为多个独立的`业务Module`，它们可以独立开发和测试，可以打包、可以同步或异步加载。**一个 Module 主要由 namespace、model、views 组成**。

- namespace 表示该 Module 的命名空间，不能重复和冲突，常与目录同名
- model 用于集中管理和操作 State
- views 跟据 State 来渲染界面

### 总结为简单四步：exportModel(), exportViews(), exportModule(), createApp()

示例一个基本目录结构如下：

```
src
├── modules  \\存放业务模块
│       ├── admin  \\一个名叫admin的业务模块
│       │     ├── views  \\存放该业务模块的视图
│       │     │     ├── Other.tsX  \\一个名叫Other的视图
│       │     │     ├── Main.tsx  \\一个名叫Main的视图
│       │     │     └── index.ts  \\导出该模块对外的视图  exportViews()
│       │     ├── model.ts  \\该模块的数据模型定义和操作  exportModel()
│       │     ├── index.ts  \\导出该模块对外的操作  exportModule()
│       │     └── exportNames.ts \\定义该模块的某些常量
│       └── app  \\一个名叫app的业务模块
│             ├── views
│             │     ├── Other.tsX
│             │     ├── Main.tsx
│             │     └── index.ts
│             ├── model.ts
│             ├── index.ts
│             └── exportActionNames.ts
└── index.tsx  \\入口文件  createApp()
```

```JS
// src/index.tsx 入口文件
import appViews from "modules/app/views";
import { createApp } from "react-coat";

createApp(appViews.Main, "root");
```

### Model 概念

> Model 为 Module 提供数据与状态的维护和更新，**主要定义 ModuleState 和 ModuleActions**

- ModuleState 表示本 Module 的状态，需要定义好数据结构和初始值
- ModuleActions 表示交互操作，分为 reducer、effect，其概念与 redux 和 saga 中的定义相同
- 原则上每个模块的 reducer 只能更新本模块的 ModuleState，但可以读取 RootState
- 支持一个 module 以观察者模式对外界的 action 进行兼听，但兼听者仍然只能修改本 module 的 State
- reducer 和 effect 只能通过 `dispatch` 方法（在 view 中）或 `put` 方法（在 model）来触发执行
- 将所有 reducer 和 effect 集中写在一个 ModuleHandlers 的 class 中
- 对外输出 ModuleActions 和 ModuleState，供外界调用
- Model 的启动过程会触发三个特定的 Action：`INIT->START->STARTED`，它们在 BaseModuleHandlers 中有默认的定义，你可以通过覆盖基类中的方法来扩展或自定义，其意义如下：
  - `INIT(): ModuleState` 它是一个 reducer，它将本模块的 initState 注入到全局 RootState 中
  - `START(): SagaIterator` 它是一个 Effect，它表示本模块正在启动，你可以在此过程中去异步拉取一些外部数据，并更新当前 State
  - `STARTED(payload: State): ModuleState` 它是一个 reducer，它表示本模块启动完毕，并更新 ModuleState，该 Action 必须在前面 `START()`Effect 中手动触发

示例一个 Model.ts 如下：

```js
// 定义该模块的State数据结构
export interface ModuleState extends BaseModuleState {
  todosList: string[];
  curUser: { //用于表示当前用户状态
    uid: string;
    username: string;
  };
  loading: { //用于表示本module中的各种loading状态
    global: LoadingState;
    login: LoadingState;
  },
}
// 定义该模块的State的初始值
const initState: ModuleState = {
  todosList: [];
  curUser: {
    uid: "",
    username: ""
  },
  loading: {
    global: "Stop",
    login: "Stop",
  },
});

// 定义该模块的ActionHandlers
class ModuleHandlers extends BaseModuleHandlers<ModuleState, RootState> {

  // 定义一个名为 login 的 effect
  // 暴露给外界使用，使用 public 权限
  @effect
  @loading("login") // 将该 effect 的 loading 状态注入 State.loading.login 中
  *login({username,password}:{ username: string; password: string }): SagaIterator {
    // 调用登录api，并获取 Resphonse
    const curUser = yield this.call(api.login, username, password);
    // 通过 this.put 触发并调用前面定义的 setCurUser
    // *** 对于 Action，包括 reducer、effet 不能用 this. 直接调用
    yield this.put(this.callThisAction(this.setCurUser, curUser));
    // 对于非 Action，可以直接调用
    this.log(username);
    // 为了方便，基类中集成了 routerActions
    // 包括 history 方法 push,replace,go,goBack,goForward
    yield this.put(this.routerActions.push("/"));
  }


  // 定义一个名为 updateTodosList 的 reducer
  // 仅内部使用，使用 protected 权限
  @reducer
  protected updateTodosList(todosList: string[]): ModuleState {
    return { ...this.state, todosList };
  }

  // 定义一个名为 updateCurUser 的 reducer
  // 仅内部使用，使用 protected 权限
  @reducer
  protected setCurUser(curUser: { uid: string; username: string; }): ModuleState {
    return { ...this.state, curUser };
  }

  // 非Action请使用 private 或 protected 权限
  private log(username: string){
    console.log(`${username} 已登录！`)
  }

  // 可以兼听另一个模块的 Action 来协同修改本模块的 State, 可以是 reducer 或 effect
  // 以观察者模式对全局的"错误 Action："@framework/ERROR"兼听，并上报后台
  // 因为兼听并不需要主动调用，请设置为 private 或 protected 权限
  @effect
  protected *[ERROR as string](payload: Error): SagaIterator {
    yield this.call(settingsService.api.reportError, payload);
  }
  // 兼听路由变化的 Action，并作出更新
  // 因为兼听并不需要主动调用，请设置为 private 或 protected 权限
  @effect
  protected *[LOCATION_CHANGE as string](payload: { location: { pathname: string } }): SagaIterator {
    if (payload.location.pathname === "/admin/todos") {
      const todos = yield this.call(todoService.api.getTodosList);
      // *** 对于 Action，包括 reducer、effet 不能用 this. 直接调用
      yield this.put(this.callThisAction(this.updateTodosList, todos.list));
    }
  }

  // 自定义启动项，覆盖基类默认的 START Effect
  // 初次进入，需要获取当前用户的信息
  @effect
  @globalLoading // 使用全局 loading 状态
  protected *START(): SagaIterator {
    const curUser = yield this.call(sessionService.api.getCurUser);
    // 必须手动触发并调用基类的 STARTED Reducer
    // *** 对于Action，包括 reducer、effet不能用 this. 直接调用
    yield this.put(this.callThisAction(this.STARTED, { ...this.state, curUser }));
  }

};
// 导出 ModuleActions
export type ModuleActions = Actions<ModuleHandlers>;
 // 创建并导出Model
export default exportModel(NAMESPACE, initState, new ModuleHandlers());
```

> ModuleHandlers 中强调与注意事项：

- 所有定义的 @reducer 和 @effect 的方法，不能直接用 this.来调用，请使用 this.put 来触发
- @effect 的方法必须显式的返回: SagaIterator
- 所有不需要被外界访问的方法都为辅助方法，请使用 private 或 protected 权限
- 如果需要自已覆盖 START 启动方法，请记得在最后手动触发 STARTED

从上面示例代码中看到，在 Model 内部，触发并调用一个 Action 必须使用`this.put`，而如果在 View 中，则需要用 `dispatch` 方法，请看示例，在模块 A 的 View 中，dispatch 模块 B 的 action：

```JS
// src/modules/A/views/Main.tsx
import B from "modules/B";

export default function(){
  return <button onClick={e => {this.props.dispatch(B.actions.logout())}}>注销</button>
}
```

### Module 路由与加载

> react-coat 中的业务 Module 是相对独立的，可以同步加载，也可以按需加载。

- 同步加载：直接引用一个 Module 的 View，会执行同步加载。 例如，模块 A 直接使用模块 B 的视图

```js
// src/modules/A/views/Main.tsx
import BViews from "modules/B/views";

export default function() {
  return (
    <div>
      <BViews.Main />
    </div>
  );
}
```

- 按需加载：使用 react-router 的方式加载。

```js
// src/modules/A/views/Main.tsx
const BView = async(() => import("modules/B/views"));
...
<Route exact path={`${match.url}/todos`} component={BView} />;
```

### Loading 机制

- loading 状态存放在每个 module 的 state 中，可以让组件绑定此状态来展示 loading UI

```js
app: {
  username: string;
  loading: {
    global: LoadingState;
    login: LoadingState;
  }
}
```

- 每个模块都有自已的一组 loading 状态，同属于一组的多个 loading 会合并，例如

```js
// 假设发起了多个异步请求，但他们可以共用一个loading状态
// 当同组内所有请求全部完成时，loading状态才Stop
setLoading(promise1, "app", "login");
setLoading(promise2, "app", "login");
```

- 每个 loading 状态有三种变化值：Start、Depth、Stop，Depth 表示深度加载，当超过一定时间，默认为 2 秒，还没有返回，则过渡为 Depth 状态

- 设置 Loading 状态有两种方法：函数方法、装饰器

- 函数方法：setLoading<T extends Promise<any>>(item: T, namespace?: string, group?: string): T;

- 装饰器方法仅用于对 effect 进行注入，@globalLoading, @moduleLoading, @loading(key="app/global")

### API

BaseModuleState, delayPromise, ERROR, getHistory, getStore, LOCATION_CHANGE, RootState, exportModule, exportViews, LoadingState, setLoading, setLoadingDepthTime, createApp, async, SagaIterator, Actions, BaseModuleHandlers, effect, exportModel, globalLoading, moduleLoading, loading, logger, reducer;

### 后记

- `使用本框架必须使用typescript吗？`

  答：推荐使用 typescript，可以做到智能提示，但也可以直接使用原生 JS

  > 欢迎批评指正，觉得还不错的别忘了给个`Star` >\_<，如有错误或 Bug 请反馈或 Email：wooline@qq.com

> [讨论留言专用贴](https://github.com/wooline/react-coat/issues/1)
