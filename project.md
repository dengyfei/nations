# 数据管理平台V2
## 一、将原先O端的代码(使用vue2.x + js)的代码移植到数据管理平台，并使用vue3 + ts + pinia 重构

## 二、点击app_id通过设定cookie值从ops.xiaoe-tools.com跳转到yarn126.xiaoe-tools.com页面并绕过登录
点击app_id 跳转到yarn126.xiaoe-tools.com页面，并且使用admin账号直接登录，由于公司内部人员都是使用admin账号登录，而且密码没有规律，很容易遗忘，因此决定直接绕过登录。

整体方案是(思路来源于同域名下的单点登录)：
* 1、进入页面，主动向后端发送请求获取cookie，然后通过 document.cookie = 'key:value; domain=.xiaoe-tools.com'
* 2、用户点击app_id，使用window.open方法打开指定路径的页面。
使用上述方法跳转并绕过登录态有几个前提：
1、在yarn126.xiaoe-tools.com页面，所有用户都是使用同一个账号密码登录，这样才能通过注入cookie的方法绕过登录。
2、我们在设置Cookie时，只能设置顶域和自己的域，不能设置其他的域。因此跳转的页面和本页面必须在同一顶域下，document.cookie的方法是BOM方法，它只能设定本域名下或者上级域名的cookie(即使它指定了domain)。当浏览器发生跳转时默认会自动携带对应域名下的所有cookie，所以如果是不同域，则跳转后不会带上刚才设定的cookie。
具体代码如下：
```vue
<script setup>
onMounted(async () => {
  // 获取cookie值
    const { data } = await getCookie();
    if (data?.data) {
      // 往对应域名上设置cookie
        Object.keys(data.data).forEach((key) => {
            document.cookie = `${key}=${data.data[key]}; domain=.xiaoe-tools.com`;
        });
    }
});
// 点击跳转
function goToYarn(data) {
    window.open(`https://yarn126.xiaoe-tools.com:30002/gateway/emr/yarn/proxy/${data.appId}/#/overview`);
}
</script>
```
三、使用vue-codemirror展示代码
vue-codemirror是一个代码展示第三方库，核心是使用了codemirror6.x。我们可以通过设定编辑器的主题、语言包和一些其他的扩展来拓展vue-codemirror的能力

四、使用Unocss的原理书写样式
UnoCSS 是一个即时原子化 CSS 引擎，或者说是一个函数式css引擎。通过预设模拟大多数已有原子化 CSS 框架的功能。
实现原理： 核心就是预置一大堆 class 样式，尽量将这些 class 样式简单化、单一化，在开发过程中，可以直接在 DOM 中写预置好的 class 名快速实现样式，而不需要每次写简单枯燥大量的 css 样式

优势：
1、按需加载，通过预扫描 --> 生成的方式实现按需加载

2、速度非常快，比Tailwind和windi快上好几百倍，原因在于Tailwind依赖于Postcss解析后的AST进行修改，Windi则是编写了一个自定义解析器和AST。而unocSS通过非常高效的字符串拼接来直接生成对应的CSS而非引入整个编译过程。同时，UnocSS对类名和生成的CSS字1符串进行了缓存，当再次遇到相同的实用工具类时，它可以绕过整个匹配和生成的过程。

3、易用，所有功能都可以通过预设和内联配置提供

和Tailwind的对比：
Tailwind:
* 工作过程：生成所有规则 ==> 扫描你代码中用到的规则 ==> 抛弃你代码中没有使用的规则
* Tailwind会依赖postcss生成的AST树

unocss:
* 工作过程：扫描你代码中用到的规则==>按需匹配并生成规则
* unocSs并不依赖AST，而是通过非常的字符串拼接来直接生成对应的cSS而不引入整个编译过程

# 服务数字化

## 一、通通知道回答的过程
用户发送问题 --> 将问题发送给后端验证有无违规词，后端是通过第三方验证 --> 验证通过后像后端发送请求，传给后端问题类型及问题内容  --> 后端调用chatGPT和文心一言，然后返回答案 --> 前端拿到答案后，调用循环，将答案一个字一个字的显示，让用户感觉像是人在回答。

## 二、在线客服

### 推荐问题
通过调用基础的接口，获取到用户目前所处板块(直播、课程、店铺、圈子等) --> 像后端获取该目录下5个常见问题，常见问题从各板块业务负责人处收集， 如果无法获取到目前所处板块或者用户当前处于首页，则随机返回5个常见问题 --> 点击某个问题，直接发送问题


### 对话框的生成