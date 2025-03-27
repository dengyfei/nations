角色控制：

vite打多包

iconfont图片引入

i18n处理日期

vue3中使用动态组件

.d.ts文件的使用

订单详情的列表怎么写的

nuxt 中  const { $axios } = useNuxtApp(); // 获取 axios 实例的意义

vite打多包，按需导出

vite-ent.d.ts的作用以及为什么在里面声明作用
```ts
/// <reference types="vite/client" />
// 为环境变量声明类型定义
interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string
}

declare module "*.vue" {
  import type { DefineComponent } from "vue";
  const component: DefineComponent<{}, {}, any>;
  export default component;
}

declare interface ApiData {
  code: number,
  msg: string,
  data: any,
}

declare interface Window {
  [key: string]: any;
}
// declare module 'vue'; 
declare module 'vue-router';
```


axios中加keepCode