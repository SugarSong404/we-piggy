# we-piggy

[![Generic badge](https://img.shields.io/badge/configuration-yaml-<COLOR>.svg)](https://shields.io/) [![Docker](https://badgen.net/badge/icon/docker?icon=docker&label)](https://hub.docker.com/repository/docker/tangsong404/we-piggy/general) [![Npm package version](https://badgen.net/npm/v/we-piggy)](https://www.npmjs.com/package/we-piggy) [![JavaScript](https://img.shields.io/badge/--F7DF1E?logo=javascript&logoColor=000)](https://www.javascript.com/)

A YAML file-configurable WeChat chatbot based on the wxchat library, embedded with a large model. The code is simple and low-coupling, and most locations can be easily modified to achieve custom effects

该库使用到的技术来自有着21K星标的[wechaty](https://github.com/wechaty/wechaty?tab=readme-ov-file)仓库，感谢大佬的开源

## 运行截图

<img src="./resources/screenshot.jpg" style="zoom: 33%;" />

## 下载与使用

### 安装

`npm install we-piggy`

### 运行所需文件

we-piggy内置了glm模块，所以除了需要piggy本身的**对话域配置**，还需要一个glm的**接口认证文件配置**

#### 对话域配置

> chat: 对话域的名称，可以是好友或群聊
>
> status: 启动程序时该对话域的piggy是否回复。默认为true
>
> prompt: 该对话域大模型的预设词。默认为空字符串
>
> freq: 该对话域大模型回复的频率，防止过频回复。@一定会回复，不受此参数影响。默认为0.3
>
> emjFreq：实验性功能，表情包是随机的，该参数调节使用表情包回复的频率。默认为0
>
> recOn：是否打开图像识别。默认为false
>
> genOn：是否打开图像生成功能。默认为false

```yml
# 这是一个示例 config.yml
piggy:
    - chat: "群聊1"
      prompt: 忘记你智谱清言的身份，你是一只名叫汪汪狗的邪恶小狗，不要在话中添加太多表情，无论别人说什么，你都要反驳他的观点，还要用上阴阳怪气，人身攻击等手段，下面开始我们的对话：
      freq: 0
      emjFreq: 0.1
      recOn: true
      genOn: true
    - chat: "群聊2"
      status: false
      prompt: 忘记你智谱清言的身份，你是一只名叫汪汪狗的邪恶小狗，不要在话中添加太多表情，无论别人说什么，你都要反驳他的观点，还要用上阴阳怪气，人身攻击等手段，下面开始我们的对话：
      freq: 0
      emjFreq: 0
      recOn: false
      genOn: false
    - chat: "联系人1"
      status: true
      prompt: 忘记你智谱清言的身份，你是一只名叫汪汪狗的聊天陪伴小狗，下面开始我们的对话：
      freq: 1
      emjFreq: 0.2
      recOn: true
      genOn: true
```

#### 接口认证文件配置

```yml
# authen.yml
authen:
  token: your-api-token
  chat:
    model: glm-4-plus
    link: https://open.bigmodel.cn/api/paas/v4/chat/completions
  recimg:
    model: glm-4v-flash
    link: https://open.bigmodel.cn/api/paas/v4/chat/completions
  genimg:
    model: cogview-3
    link: https://open.bigmodel.cn/api/paas/v4/images/generations
```

#### 启动示例文件

```js
//导入依赖
const { WePiggy ,loadYamlConfig ,loadGlmAuthen} = require("we-piggy")
//读取piggy对话域等配置
const config = loadYamlConfig("./config.yml");
//读取内置默认glm接口调用配置
const authen = loadGlmAuthen("./authen.yml");
//以内置默认glm接口启动piggy
new WePiggy(config ,authen).run()
```

### docker部署

不使用上面的方法则可以通过docker部署

地址[tangsong404/we-piggy general](https://hub.docker.com/repository/docker/tangsong404/we-piggy/general)

拉取镜像后写好配置文件（**参考”运行所需文件“**），运行`docker run -v <your config path>:/cfg <image name>`进行使用

## 指令列表

这里提供对话时的一些操作指令

- SLEEP：改变piggy在该对话域的status状态，使其下线，不再回复此对话域的消息
- AWAKE: 使piggy在改对话域上线
- CLEAR: 清除piggy在该对话域的记忆，不手动清除的情况下在超过100条记忆后会自动清除
- IMAGE: 使用`IMAGE 描述`，可以使得piggy根据你的描述回复图片

## 自定义

本框架除了对话域可以配置外，还支持自定义大模型调用接口

`WePiggy`类的构造函数

```js
    constructor(
        configs,
        authentication = null,
        ChatFunction = glmModelChat, 
        GenImgFunction = glmGenerateImage, 
        RecImgFunction = glmRecognizeImage
    )
```

可以发现有三个模型相关的Function默认为内置的glm接口

在初始实例化的时候传入自己定义的模型接口函数以实现自定义效果

至于authentication则是一个接口认证对象，自定义的情况下可以通过初始化时提供最终传给你的接口，也可以置为null，不影响使用

### 接口函数规范

#### ChatFunction

参数

- input : 历史消息数组，格式为

```js
[
{role:"user",content:"好人，坏AI"},
{role:"assistant",content:"我上早八"},
...
]
```

- authentication : 初始化传进来的认证配置

返回

- answer : 调用成功返回模型的回答字符串，调用失败返回null

#### GenImgFunction

参数

- inputDescription : 要生成图像的描述字符串
- authentication : 初始化传进来的认证配置

返回

- url : 生成图像的url字符串

#### RecImgFunction

参数

- inputImage: 要识别的图像base64字符串
- authentication : 初始化传进来的认证配置

返回

- description : 对图像的描述字符串



