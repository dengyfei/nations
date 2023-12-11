pinia 使用
vueUse 使用
事件总线型的 hooks 和 vuex/pinia 的区别
了解乾坤用法，
npm 包开发流程中 loading 的关闭
sense
toRef,toRefs

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
