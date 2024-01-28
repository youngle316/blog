---
title: Unsplash API Production 申请注意事项
publishedAt: 2023-10-04
summary: 文章主要介绍了在申请 Unsplash API 时遇到的问题，并如何解决的
cover: https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/2.png
href: unsplash-api-production-application-guidelines
---
![2.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/2.png)

最近搞了个封面图制作工具 [Cover Paint](https://coverpaint.xiaole.site/)，使用 Unsplash 作为背景图，由于 Demo 版本每小时只能访问 50 次并不够用，所以需要申请 Production 版本（每小时 5000 次请求）。

由于一开始没有细看申请的要求导致审核了有三四次才通过，这边文章就记录一下具体的过程。

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20231004124102.png)

## 创建 App

1. 进入 [Unsplash Image API](https://unsplash.com/developers)官网，注册登陆并点击 `Your apps` 按钮
2. 点击 `New Application` 创建一个新的应用
3. 同意所有条款，输入应用名称和描述
4. 至此，新的 Demo 版本的应用已经创建好了

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20231004115950.png)

## 使用 Unsplash api

在前端我使用的是 [unsplash-js](https://github.com/unsplash/unsplash-js) 这个官方提供的库，具体使用的方法可以看官方的文档。

其中需要的 key 可以在上一步创建的应用界面找到

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20231004120530.png)

## 申请 Production 版本

可以提前查看 [Unsplash API Guidelines](https://help.unsplash.com/en/articles/2511245-unsplash-api-guidelines)。建议先将自己做的网站或别的类型的产品 Unsplash 相关部分先做完再进行申请，每小时 50 次是够开发用的。因为申请应该是需要提交代码截图的。需要注意的是**产品的名称不要和 Unsplash 雷同**。

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20231004121452.png)
### 添加图片作者的信息

每一张图片底部或其他位置都需要添加醒目的所属作者的信息，点击作者名称可以跳转到对应的 Unsplash 摄影师详情页（图片的所有信息包括摄影师等 `unsplash-js` 已经提供了）。

注：这里需要上传一张截图（最好是提供视频）。如果是 web 应用，需要鼠标放在摄影师名字链接上，浏览器左下角会出现跳转的链接地址。如果是其他类型的应用，则需要添加代码截图，表示点击摄影师名称会跳转到对应的网页。并且链接必须添加 `utm` 参数。

如下：其中 `utm_source` 可填写为创建的应用名称

```html
<a  
  target="_blank"  
  href={`https://unsplash.com/@${user.username}?utm_source=cover-paint&utm_medium=referral`}  
>  
  {user.name}  
</a>
```

### 追踪下载

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20231004123833.png)

如果下载量为0，则审批不会通过。

已我的项目举例，当点击图片并添加了文字信息后。点击下载时，需要追踪这个下载的行为，unsplash-js 中提供了这个 api，[unsplash-js-downloadtrack](https://github.com/unsplash/unsplash-js#photostrackdownloadarguments-additionalfetchoptions)。当下载完成时调用这个 api 即可。

这里可以提供一张截图，并且在开发时就需要触发这个追踪下载行为，因为下载这个数据可能需要等到第二天才会有数据显示。

```typescript
const downloadCover = async () => {
    const ref = coverRef?.current as HTMLDivElement;
    if (ref === null) {
      return;
    }
    setDownloading(true);

    toPng(ref, { cacheBust: true })
      .then((dataUrl) => {
        const link = document.createElement("a");
        link.download = `coverImage.png`;
        link.href = dataUrl;
        link.click();
      })
      .catch((err) => {
        console.log(err);
      })
      .finally(() => {
        setDownloading(false);
      });

    if (unsplashInfo) {
      await trackDownload(unsplashInfo.downloadLocation);
    }
  };
```

### 添加应用描述信息

最后一步需要在应用界面填写完整详细的应用描述，并尽可能的提供项目的截图或视频。你要做一个什么应用，在项目中使用 Unsplash API 做什么。如果有可能可以提供一个项目 demo 地址。

如图我是这样填写的。
![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20231004124454.png)
通过上述几步应该可以申请到 Production，并且审核的时间还挺快的（48小时之间一般都是有回复的）。如果申请不通过可根据邮件提示内容再继续补充所需资料。
