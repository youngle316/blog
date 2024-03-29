---
title: 如何在 Next.js 项目中使用 supabase
publishedAt: 2023-08-16
summary: 文章主要介绍了 supabase的基本情况以及如何在 next.js 项目中使用
href: how-to-use-supabase-in-a-nextjs-project
cover: https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/blog-5.png
---
![blog-5.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/blog-5.png)

## 什么是 supabase

[supabase](https://supabase.com/) 是一个替代 Firebase 的开源 BaaS 项目。其提供了各种现代应用所需的稳定、强大、易于使用和可扩展的后端功能。Supabase 基于 PostgreSQL 数据库和 RESTful API 构建，并提供身份验证、实时数据推送、存储和其他一些常见的后端服务。

不需要买服务器，只需掌握简单的 SQL 语句和数据库知识就可以创建一个后端服务。当然它的功能是非常强大的，这篇文章只介绍如何在 Next.js 中使用。

## 使用 supabase

>学习任务内容都需要详细的阅读官方文档

根据官方文档 [Use Supabase with NextJS](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs) 这一节的内容，使用步骤分为如下几步

这里我使用 supabase 存储博客的阅读量

### 创建项目并建表建数据

通过 [https://app.supabase.com/projects](https://app.supabase.com/projects) 创建一个项目

![new project](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/202304281348913.png)

通过 SQL 编辑器或图形界面 DataBase 创建一个表名为 Views

![database](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/202304281355938.png)

这里通过图形界面创建，在 Database 中使用右上角的 New table 创建 Views 表

![create table](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20230428140235.png)

列中的 slug 表示博客的 title，count 表示对应博客的访问量

### 创建 Next.js 项目

```bash
npx create-next-app@latest --typescript
```

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20230428141104.png)

### 安装 supabase-js

```bash
npm install @supabase/supabase-js
```

### 创建 supabase

在根目录下创建 lib 文件夹，创建一个名为 supabase.ts 的文件

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL as string;
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_KEY as string;
export const supabase = createClient(supabaseUrl, supabaseKey);
```

project_url 和 supabase_key 可以在项目的设置选项中找到

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20230428142236.png)

同样不建议将 key 暴露出去，可以存在 env.local 中

>💡 tips: 在 Next.js 中使用 env.local，需要加上 NEXT_PUBLIC 前缀，否则会报错。

### CRUD

详细的命令可以通过 [JavaScript](https://supabase.com/docs/reference/javascript/installing) 文档查询

当打开博客文章页面时，需要查询当前博客的阅读量，可以使用

```typescript
fetch(`/api/views`);
```

在 pages/api/views 目录下创建 index.ts

```typescript
import { supabase } from '@/lib/supabase';
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    let { data } = await supabase.from('Views').select('*');
    return res.status(200).json(data);
  } catch (e: any) {
    console.log(e);
    return res.status(500).json({ message: e.message });
  }
}
```

通过写类似于 sql 语句，可以从 Views 表中获取全部的数据

也可以通过 post 传参将 slug 传进来，查询对应的数据

同时当打开博客文章页面时，也需要对当前博客的阅读量加一，就可以用过 post 请求将 slug 参数传入

```typescript
fetch(`/api/views/${slug}`, {
	method: 'POST'
});
```

在 pages/api/views 目录下创建 `[slug].ts`

```typescript
import { supabase } from '@/lib/supabase';
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const slug = req.query?.slug as string;
    if (!slug) {
      return res.status(400).json({ message: 'Slug is required.' });
    }

    const { data } = await supabase
      .from('Views')
      .select('count')
      .eq('slug', slug);

    const views = !data?.length ? 0 : Number(data[0].count);

    if (req.method === 'POST') {
      if (views === 0) {
        await supabase.from('Views').insert({ count: views + 1, slug: slug });

        return res.status(200).json({
          total: views + 1
        });
      } else {
        await supabase
          .from('views')
          .update({ count: views + 1 })
          .eq('slug', slug);
        return res.status(200).json({
          total: views + 1
        });
      }
    }

    if (req.method === 'GET') {
      return res.status(200).json({ total: views });
    }
  } catch (e: any) {
    console.log(e);
    return res.status(500).json({ message: e.message });
  }
}
```

新建使用 insert，更新使用 update。这样就可以将阅读量存储到 supabase 中

## 其他功能

同时 supabase 也支持 bucket 存储功能。我的项目中将 og image 存储在 supabase 中

![image.png](https://obsidian-picgo-le.oss-cn-hangzhou.aliyuncs.com/img/20230428145812.png)

准备后面的项目中继续使用 supabase，替代 Firebase。
