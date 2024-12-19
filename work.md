# 调用灰网接口

- 1、从现网中获取 cookie
- 2、 获取灰度名单
- 3、在现网 cookie 后面拼上`app_id`和`with_app_id`,值均为灰度名单

# 环境名称

- 现网：上线的环境
- 内灰：公司内部测试使用
- 外灰：允许部分用户使用，体验版
  內灰和外灰其实都是现网环境，实际上没有本质的区别。

# 独立车构建流程

1、创建工单
2、生成上线计划
3、从计划列表中点击名称进入计划详情页
4、代码合并中拉取分支、发起合并请求
5、到 gitlab 中同意合并
6、打 tag，进行构建，代码发布中可以查看进程
7、点击发布
8、点击名单发布中的`查看|编辑名单`
9、点击名单检索，将 appid 输入到店铺名单中，点击开始检索并添加名单
10、点击发布名单
11、访问，从现网中访问，并加上`app_id`和`with_app_id`
12、注：前后端代码需在同一班车上，如果是后加的系统，需要在计划详情页面点击`计划工单` --> `重新生成步骤`

# 小车构建流程

1、创建工单
2、生成上线计划
3、从计划列表中点击名称进入计划详情页
4、放到开发环境中。具体如下 进入`产研协助平台开发环境` --> 点击`开发平台` --> `CDN灰度发布`中搜索对应系统和版本。注意版本需要是 feature 分支。
5、开发环境中验证通过后，发起合并到 master 请求
6、发布到准线网(发起合并，打tag，加店铺名单)
7、准现网验证通过后发布到现网

# 侧边栏高亮流程

首先需要接入公共模板，因为基本流程都是由公共模板完成。
1、首先 sam 哥会在公共模板中注入一个变量，基础平台会根据这个变量请求翔哥接口，获取该一级目录下所有三级目录的`highlightId`，
2、然后业务侧负责高亮显示，在`app.vue`中通过调用`GlobalState.set({highlightId: value})`,其中 value 是翔哥那边的 highlightId，具体代码如下：

```js
watch: {
    $route: {
        handler(newval, oldval) {
            // 滚回顶部
            document.querySelector('#data-container')?.scrollIntoView();
            // 高亮菜单
            const highId = newval.name || oldval.name;
            window.GlobalState.set({
                highlightId: highId
            });
        },
        // 深度观察监听
        deep: true
    }
},
```
所以，我们每次在菜单栏中新增一个tab，都需要翔哥那边操作，并告诉他highlightId值

# 微组件使用
如果微组件项目组织结构如下：
```js
├─ai-assistant-widget
    ├─libCos
    │  ├─eai-assistant-widget
    │  └─eai-assistant-widget1
```
```js
MircoComponent = await Mimir({ section: 'ai-assistant-widget', project: 'project_name' })?.get('eai-assistant-widget');
```

其中`section`表示目录名，是libCos的父级目录名称，`project`代表唯一key，一般为项目名，而且和班车中绑定的配置项系统名要相同，get后的参数为libCos打包后的名字，是libCos的子目录名称
班车中配置的系统名对应为project，增加的配置项(微组件名)对应为section。

# 查看店铺是否在灰度中
产研平台 ==》 运维平台 ==》 服务搜索 ==》店铺灰度查询


# cdn发布
1、前端输出成果物结构须符合公共模板要求
2、成果物上传到cos桶
3、找运维把域名指到公共模板服务（独立域名才需要，统一域名无法找运维）
4、找sam哥注册对应系统，注册后返回的公共模板就能返回对应系统的静态资源了。 


# sam哥路径配置
1、找sam配置路径，路径需要符合规范
2、将router下的base配置成给sam哥的路径

# 组件发布
1、班车系统 --> 服务管理 --> 服务列表 --> 添加系统。添加完系统后，每次提交都会自动创建githook，然后触发coding构建
2、在项目中新建build.yaml文件

# 广告投放
1、clickId由广告平台生成，用户每一次进入广告落地页的clickId都不一样，因此可视为唯一值。
2、clueId--线索id， 由srcm生成，有三个时间生成：
    2.1、有用户的下单场景，即非微信环境下留资购课加微场景，当填写信息采集后，由scrm生成。
    2.2、加微后生成，此时可以得知用户，因此scrm可以生成
    2.3、登录小程序后生成，此时可以得知用户，因此scrm可以生成
3、userId： 用户提交信息采集后，由信息采集组件生成。
4、linkUserId: 用于锁课锁码，同一用户不同终端，或同一用户不同应用均不一样；匿名用户下单，使用link_user_id+linkId判断权益

## 问题
1、小程序无法展码
调用 update_click_record 的时候接口没有返回clue_id，还继续往下执行。

2、支付后未展码，跳转到其他地方

自定义域名由于跨域问题，无法直接拉起支付，所以在jinwei系统中，做了一个判断，如果是独立域名时，前端直接跳转到域名为h5.xiaoe.com的新路径，这个路径是一个空白页，在这个空白页拉起支付，已解决跨域导致的无法拉起支付问题。
在空白页支付完成后，会进行跳转，跳转到后端返回的一个redirect_url页面，如果后端没有返回redirect_url，则默认跳转到上一个reference页面，即h5.xiaoe.com(店铺首页)。
因此，如果支付完成后跳转到首页的原因就是后端没有返回一个redirect_url的字段