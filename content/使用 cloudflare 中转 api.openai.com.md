---
title: 使用 cloudflare 中转 api.openai.com
publishedAt: 2023-08-29
summary: 文章主要介绍了两种方式解决 openai 在特定地区无法使用的问题
cover: https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/blog-4.png
href: use-cloudflare-as-a-proxy-for-api-openai-com
---
![blog-4.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/blog-4.png)

最近做了一个 [ChatGPT](https://github.com/youngle316/power-chatgpt) 项目，想分享给家人使用，由于在国内不能直接访问 openai api，所以需要对官方的接口进行代理。

方法参考 [chatgptProxyAPI](https://github.com/x-dr/chatgptProxyAPI) 中提供的方式。

### 使用 cloudflare pages

1. 打开 GitHub，创建一个仓库。仓库名随便填

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230830164206.png)
2. 创建一个新的文件

![](https://raw.githubusercontent.com/youngle316/picg/main/20230830164308.png)

文件名为 `_worker.js` 。如果想代理其他地址，可以直接修改 `TELEGRAPH_URL`

内容为:

```javascript
const TELEGRAPH_URL = 'https://api.openai.com';


export default {
  async fetch(request, env) {
      const NewResponse = await handleRequest(request)
      return NewResponse
  },
};

async function handleRequest(request) {
  const url = new URL(request.url);
  const headers_Origin = request.headers.get("Access-Control-Allow-Origin") || "*"
  url.host = TELEGRAPH_URL.replace(/^https?:\/\//, '');
  const modifiedRequest = new Request(url.toString(), {
    headers: request.headers,
    method: request.method,
    body: request.body,
    redirect: 'follow'
  });
  const response = await fetch(modifiedRequest);
  const modifiedResponse = new Response(response.body, response);
  // 添加允许跨域访问的响应头
  modifiedResponse.headers.set('Access-Control-Allow-Origin', headers_Origin);
  return modifiedResponse;
}
```

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230830164656.png)


然后 commit changes

3. 打开 cloudflare 创建一个新的 pages，并连接自己的 Git

![](https://raw.githubusercontent.com/youngle316/picg/main/20230830165049.png)

4. 选择刚才创建的仓库
5. 使用默认的配置即可（不用任何修改，直接进行下一步部署）
6. 等待部署完成，返回到创建的 pages 页面，就可看到地址

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230830165420.png)
#### 使用方式：
>如何在我的项目中使用

1. 打开 `https://power-chat.younglele.cn`  
2. 在设置 `OpenAI API Proxy` 中填写 `https://上面获取到的域地址/v1`

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230830171641.png)

3. 保存即可开始无需翻墙访问 `openai api`

>还有一种使用 cloudflare worker 方式，但是需要有自己的域名并绑定到 cloudflare 上，还是推荐使用 page 的方式

### 使用 Cloudflare Worker

1. 在 Workers 和 Pages 中创建应用程序

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230829175343.png)

2. 点击 创建 Worker

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230829175601.png)

3. 修改名称并点击部署（代码部署后再修改）

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230829175812.png)

4. 点击编辑代码

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230829175936.png)

将下面的代码粘贴到代码编辑器中，点击保存并部署

```javascript
const TELEGRAPH_URL = 'https://api.openai.com';

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url);
  const headers_Origin = request.headers.get("Access-Control-Allow-Origin") || "*"
  url.host = TELEGRAPH_URL.replace(/^https?:\/\//, '');
  const modifiedRequest = new Request(url.toString(), {
    headers: request.headers,
    method: request.method,
    body: request.body,
    redirect: 'follow'
  });
  const response = await fetch(modifiedRequest);
  const modifiedResponse = new Response(response.body, response);
  // 添加允许跨域访问的响应头
  modifiedResponse.headers.set('Access-Control-Allow-Origin', headers_Origin);
  return modifiedResponse;
}
```

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230829180101.png)

成功后返回创建的 workers 页

这样做了之后会发现还是没有好，是因为 workers.dev 仍然是被墙的。这时就需要绑定一个域名

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230830162238.png)

但是这个自定义域现在只支持 Cloudflare 上处于活动状态的域，需要点击左侧的 **网站** 按照步骤添加一个自己的域名或在 cloudflare 购买

![image.png](https://raw.githubusercontent.com/youngle316/picg/main/20230830162132.png)


