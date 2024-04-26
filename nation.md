pinia 使用
vueUse 使用
事件总线型的 hooks 和 vuex/pinia 的区别
了解乾坤用法，
npm 包开发流程
vue3中挂载vue2组件
.d.ts文件

vueuse中的useEventSource
AbortController
EventSource


# 事件总线

在 vue2 中有事件总线这一概念，在 vue3 中摒弃了这一概念，其原因是引入了 hooks。也就是说，我可以把多个组件公用的属性抽取到一个 ts 文件中，并且可以在这个文件中定义修改该变量的方法，然后将属性和方法暴露出去。在需要用到的地方进行引入即可。

# InstanceType

**作用：**
InstanceType 用于构造一个由 Type 中构造函数的实例类型组成的类型，即，给定一个构造函数类型 T，InstanceType 返回的是构造函数生成实例的类型 T

在 vue 中需要 InstanceType<typeof Component>得到组件实例类型，而不是直接用 Component 作为类型，是因为 Component 不是 class，它只是一个构造函数，他没有声明实例类型。
**用法：**
InstanceType<Type>

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }
}

const Jack: InstanceType<Person> = new Person('jack')
```

# 滚动到某个位置

```js
/**
 * 将某个元素滚动到顶部
 * 原理是触发父级组件滚动，使得该元素位于父元素最顶部，因此要求父元素必须有滚动条
 */
document
  .querySelector('#data-container')
  ?.scrollIntoView({ behavior: 'smooth' })

/**
 * 将某个元素滚动到父元素底部，或中间
 * 原理同上
 */
document
  .querySelector('#data-container')
  ?.scrollIntoView({ block: 'start/center/end', behavior: 'smooth' })
```

# vue 的内置组件 component 中的 is 属性需要传入的是组件的名称，而不是一个简单的字符串

# vue-router 提供的 hooks，比如 useRouter 等等，都是只能在 setup 函数中使用，而且不能在自定义函数中使用

# toRef 和 toRefs

## toRef

针对一个响应式对象，创建一个具有响应式的 ref 对象，并且两者间仍保持引用关系。

```vue
<script setup>
const state = reactive({
  age: 23,
})

const ageRef = toRef(state, 'name')
</script>
```

上面的代码利用 toRef 生成的 ageRef 也是一个 ref 对象，并且和 state 中的 age 属性保持引用关系，即任何一个改变都会影响另一个。

## toRefs

将一个响应式对象转为普通对象，对象的每一个属性都是对应的 ref，并且两者保持引用关系。

```vue
<script setup>
const state = reactive({
  age: 23,
  name: 'jack',
})

const stateRefs = toRefs(state)
console.log(stateRefs.name.value)
</script>
```

上面的代码中，stateRefs 是一个普通对象，对象的每一个属性都是一个 ref 对象，并且和原来的保持引用关系，即，任何一个改变都会影响另一个。但是 toRefs 只能转换第一层的属性。

正因为 toRefs 返回的是一个普通对象，所以可以使用解构，而且解构出来的也是一个个的 ref 对象。

## 共性和区别

共性： 两者都是将 reactive 属性转换成 ref 对象，并和原来属性保持引用关系。两者都不会创造响应式，只延续响应式。
区别：toRef 是指定某一节点提取出来，toRefs 是一次性将所有节点提取出来。但 toRefs 只能提取一级节点。

# element plus 自定义命名空间

1、设置 scss 和 css 变量

```scss
// styles/element/index.scss
@forward 'element-plus/theme-chalk/src/mixins/config.scss' with (
  $namespace: 'ep'
);
```

这样配置后，从 element plus 中导入的 scss 文件中的`$namespace`变量都是`ep`了。

2、配置 vite.config.js，主要实现以下效果：

- 按需导入 element plus 组件
- 从 element plus 中导入 scss 文件
- 将刚刚配置的 scss 文件作为全局配置

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import path from 'path'

export default defineConfig(({ command, mode }) => {
  return {
    plugins: [
      vue(),
      // 按需导入element plus组件
      Components({
        resolvers: [
          ElementPlusResolver({
            importStyle: 'sass', //导入scss文件
          }),
        ],
      }),
    ],
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'src'),
      },
    },
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `@use "@/assets/element.scss" as *;`, //设置全局样式文件配置
        },
      },
    },
  }
})
```

3、设置 ElConfigProvider

使用 ElConfigProvider 包装您的根组件。

```vue
<!-- App.vue -->
<template>
  <el-config-provider namespace="ep">
    <el-button type="primary">Primary</el-button>
  </el-config-provider>
</template>
```

如此一来，生成的元素的类名就不再是`el-*`，而是`ep-*`，比如上面的按钮的类，最终是`ep-button`，这样就可以通过自定义命名空间的方式来避免样式冲突。
此时，会导致 MessageBox 及 Message 的默认样式无法生效。
此时我们需要在之前的 sass 文件中单独引入两个组件的样式文件即可

```scss
@forward 'element-plus/theme-chalk/src/mixins/config.scss' with (
  $namespace: 'eai'
);
@use 'element-plus/theme-chalk/el-message.css';
@use 'element-plus/theme-chalk/src/message-box.scss';
```
除了手动引入样式，我们还可以配置自动导入样式：

```js
pnpm add vite-plugin-style-import -D
```

```js
//vite.config.ts

import {
  createStyleImportPlugin,
  ElementPlusResolve,
} from 'vite-plugin-style-import'

export default defineConfig({
  plugins: [
    createStyleImportPlugin({
      resolves: [ElementPlusResolve()],
      libs: [
        // 如果没有你需要的resolve，可以在lib内直接写，也可以给我们提供PR
        {
          libraryName: 'element-plus',
          esModule: true,
          resolveStyle: (name: string) => {
            return `element-plus/theme-chalk/${name}.css`
          },
        },
      ],
    }),
  ],
})
```

# composition API 的优势

* 极易复用，由于composition API的方法都是以JS原生函数的形式呈现，所以极大的提高了可复用性，甚至可以抽离成hooks函数被多个组件共同使用
* 生命周期函数可以多次使用，灵活组合
* 提供更好的上下文支持，在options API中经常会由于this上下文指向不明确而导致一些难以预料和排查的问题，比如mixin。
* 更好的Typescript类型支持
* 可以按功能和逻辑组织代码，提高可维护性。比如，我们可以登录功能的所有逻辑放在一起，也可以抽离单独的hooks文件。