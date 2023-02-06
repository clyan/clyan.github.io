## 背景

web端实现原生侧滑动画

## 注意点

缺点：浏览器环境中页面刷新后动画会混乱，App中不会（因为App中不会提供刷新整个系统的机制，最多是提供刷新当前页的数据）

踩坑：安卓App中，尽量使用第三方Webview提高兼容性（使用自带的webview部分机型会卡顿）、避免与vconsole一起使用（vconsole的层级会影响动画的过渡，导致卡顿）

## 现有方案

搜索了大量的资料，大致包括以下两种方式：

1. 使用`route.meta`手动为每个路由添加index维护层级， 参考实现：[vue 路由切换动画](https://blog.csdn.net/m0_61437084/article/details/121037559)

2. 使用`sessionStorage`自动为访问过的路由存储index至缓存维护层级(相对)，参考实现：[Vux UI](https://github.com/airyland/vux/blob/v2/src/main.js)

以上两种方案，给每个路由维护一个路由层级index，如：原页面`index = 1`跳转至目标页面`index = 2`， `1 < 2` 则应用`右 -> 左`滑动，`2 > 1` 则应用 `左 -> 右`。

**缺点：** 维护层级繁琐、路由跳转时会导致路由动画混乱，假设有如下情况，有两条不相关的路由层级，产品相关的路由层级如下`A1 -> A2 -> A3 -> A4`, 用户路由层级如下 `B1 -> B2 -> B3 -> B4`。首先假设已经进入`产品A3`页面，此时需要跳转到`用户B2`页面, 那么此时应该应用的动效是`In动画(左 -> 右)`，但是由于`B2 < A3`,按照维护层级的思路，实际却应用了`Out动画(右 -> 左)`。

## 换个思路

不维护任何路由层级，思考vue中使用`vue-router`路由跳转有哪几种方式，使用router的方法，`go、push、replace、back、go(- | 0 | +)`。那么我们只需要将路由前进与与路由后退归成两类，前进时应用`In动画(左 -> 右)`， 后退时应用`Out动画(右 -> 左)`。

**实现该动效果有以下几个步骤**

1. 添加动画所需CSS

2. 劫持路由跳转方法，记录此次跳转应该应用的动画

3. 避免IOS自带侧滑动画冲突（在第2步中处理）

4. 使用第2步记录的动画

以下假设您的项目`<router-view></router-view>`在App.vue中。

### 添加动画CSS
在App.vue中添加如下CSS。

```css
.page-out-enter-active,
.page-out-leave-active,
.page-in-enter-active,
.page-in-leave-active {
  will-change: transform;
  transition: transform 0.25s ease-out;
  height: 100%;
  width: 100%;
  top: 0;
  left: 0;
  position: fixed;
  backface-visibility: hidden;
  perspective: 1000;
  background-color: #ffffff; // 一般应用为项目中的背景色
}

.page-out-enter-from {
  transform: translateX(-30%);
}

.page-out-leave-active {
  transform: translateX(100%);
  z-index: 2;
}

.page-in-enter-from {
  transform: translateX(100%);
}

.page-in-leave-active {
  transform: translateX(-30%);
}
</style>
```
### 劫持路由跳转方法

在`router/index.ts`中添加如下代码：

```js
import { createRouter, createWebHistory, RouteRecordRaw } from "vue-router";
import { routeTransition } from "./router-helper";
const routes: Array<RouteRecordRaw> = [
  {
    ...详细路由配置...
  },
];

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes,
});

// 应用路由跳转动画
routeTransition(router);
```

在`router/router-helper.ts`中添加如下代码：

**注：** 劫持路由跳转方法是为了在函数调用前执行额外的操作，也就是一个切面的操作，单独抽离`inject函数`放到`@/utils/aop`中, 同时`isIosJump`是为了解决IOS的侧滑动画与自定义动画冲突。

```js
import { Router } from "vue-router";
import { inject } from "@/utils/aop";

export enum DirectionType {
  // 进入动画
  In = "in",
  // 退出动画
  Out = "out",
  /** 不应用动画，用于处理IOS侧滑冲突 */
  None = "",
}

// 路由动画
export const routeTransition = (router: Router) => {
  /** 默认认为是IOS的侧滑返回，通过监听router方法进行改变 */
  let isIosJump = true;
  
  // 默认不应用动效
  let direction = DirectionType.None;

  /** 处理路由跳转动画，push、replace、go(正值｜0)应用In动画，其他情况应用Out动画 */
  router.replace = inject("before", router.replace, () => {
    // 只要是调用了路由调转方法就认定不是IOS默认侧滑返回
    isIosJump = false;
    
    // 替换路由时，应用 `In动画(左 -> 右)`
    direction = DirectionType.In;
  });

  router.push = inject("before", router.push, () => {
    isIosJump = false;
    
    // 添加路由时，应用 `In动画(左 -> 右)`
    direction = DirectionType.In;
  });

  router.go = inject("before", router.go, (delta) => {
    isIosJump = false;
    
    // 注意：delta=0时，也可单独判断，设置direction 为 DirectionType.None，不应用动画，因为相当于当前页面刷新
    if (delta >= 0) {
      // 路由前进时，应用 `In动画(左 -> 右)`
      direction = DirectionType.In;
    } else {
      // 路由回退时，应用 `Out动画(右 -> 左)`
      direction = DirectionType.Out;
    }
  });

  router.back = inject("before", router.back, () => {
    isIosJump = false;
    
    // 路由回退时，应用 `Out动画(右 -> 左)`
    direction = DirectionType.Out;
  });

  router.beforeEach(function (to, from, next) {
    // 如果是IOS侧滑则不应用侧滑动效
    if (isIosJump) {
      direction = DirectionType.None;
    }
    // 记录动画滑动方向存放在，目标页面route.meta中
    to.meta.direction = direction;

    next();
  });

  router.afterEach(() => {
    isIosJump = true;
  });
};
```
`@/utils/aop.js`代码如下：
```
export type InjectPosition = "before" | "after";

export function inject<T extends (...args: any) => any>(
  position: InjectPosition,
  func: T,
  handler?: (...args: Parameters<T>) => void
): T {
  return ((...args: Parameters<T>): ReturnType<T> => {
    if (position === "before") handler?.(...args);
    const result = func(...(args as []));
    if (position === "after") handler?.(...args);
    return result;
  }) as T;
}
```

经过以上劫持路由方法处理，此时应该应用的动效`direction`已存在目标路由的meta中。

### 使用动画
在`App.vue`中添加如下代码

**注：** 为了不使`App.vue`中的逻辑过多，抽取`useDirection hook`获取应用的动画

```js
<template>
  <router-view v-slot="{ Component }">
    <transition :name="direction" :css="!!direction">
      <keep-alive>
        <component :is="Component" />
      </keep-alive>
    </transition>
  </router-view>
</template>
<script lang="ts">
import { useDirection } from "@/hooks/useDirection";
export default {
  name: "App",
  setup() {
    const direction = useDirection();
    return {
      direction,
    };
  },
};
</script>
```

`@/hooks/useDirection` 代码如下:
```
import { computed } from "vue";
import { useRoute } from "vue-router";
import { DirectionType } from "@/router/router-helper";

export const useDirection = () => {
  const route = useRoute();
  const direction = computed(() => {
    // 获取当前页面滑动动效方向， "" 表示不应用动效，处理ios下动效冲突问题
    if (route.meta.direction === DirectionType.None) return DirectionType.None;
    // 与动画名字保持一致
    return (
      "page-" +
      (route.meta.direction === DirectionType.In
        ? DirectionType.In
        : DirectionType.Out)
    );
  });
  return direction;
};

```

至此一个IOS侧滑动效就完成了，完整的Demo地址如下： [vue3.0-router-transition](https://github.com/ywymoshi/vue3.0-router-transition)