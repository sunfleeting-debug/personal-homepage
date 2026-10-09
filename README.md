<div align="center">
  <img src=".github/assets/icon.png" width="108" alt="personal-homepage" />
  <h1>personal-homepage</h1>
  <p><b>启明星-长庚星博客的评论后端</b><br /><sub>一个部署在 Vercel 上的 Waline 服务</sub></p>
  <p>
    <a href="https://personal-homepage-phi-eight.vercel.app"><img src="https://img.shields.io/badge/live-Vercel-000000" alt="Live on Vercel"></a>
    <img src="https://img.shields.io/badge/Waline-%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F-00B96B" alt="Waline">
    <img src="https://img.shields.io/badge/PostgreSQL-%E5%AD%98%E5%82%A8-4169E1" alt="PostgreSQL">
    <img src="https://img.shields.io/badge/Node.js-%E2%89%A518-339933" alt="Node >= 18">
  </p>
  <p>
    <b>简体中文</b> ·
    <a href="README.en.md">English</a>
  </p>
</div>

> ⚠️ **仓库名有误导性**：这里**不是**一个个人主页，而是给博客 [启明星-长庚星](https://sunfleeting-debug.github.io/) 用的**评论服务端**。
> 名字是历史遗留（当初想把 Waline 部署在 `personal-homepage` 这个仓库下），内容一直是 Waline。

## 一览

| | |
| --- | --- |
| **它是什么** | [Waline](https://waline.js.org/) 评论系统的服务端，跑在 Vercel Serverless 上 |
| **线上地址** | <https://personal-homepage-phi-eight.vercel.app> |
| **谁在用** | 博客 `valaxy/demo/yun/site-meta.ts` 里的 `WALINE_SERVER_URL` 指向它 |
| **存储** | PostgreSQL（`POSTGRES_*` 环境变量） |
| **构成** | 3 个有效文件：`index.cjs` · `vercel.json` · `robots.txt` |
| **索引** | `robots.txt` 全站 `Disallow: /` —— 评论接口不该被搜索引擎收录 |

---

## 📁 这个仓库里有什么

```text
.
├── index.cjs         Waline 应用入口（可挂插件、可写 postSave 钩子）
├── vercel.json       构建与路由：除 robots.txt 外全部重写到 index.cjs
├── robots.txt        User-agent: * / Disallow: /  —— 不索引
├── package.json      唯一依赖 @waline/vercel
└── .env.example      PostgreSQL 连接配置模板
```

`index.cjs` 全部内容就这几行 —— 评论的存储、鉴权、管理后台都由 `@waline/vercel` 提供，这里只负责把它挂起来：

```js
const Application = require('@waline/vercel');

module.exports = Application({
  plugins: [],
  async postSave(comment) {
    // do what ever you want after comment saved
  },
});
```

`vercel.json` 里有两处值得注意：

- **`rewrites`**：`/((?!robots\.txt$).*)` → `index.cjs`，即除了 `robots.txt` 之外**所有路径**都交给 Waline 处理（它自己带管理后台与 API 路由）。
- **`includeFiles`**：显式带上 `@mathjax/mathjax-newcm-font`、`mhchemparser`、`ip2region/data` —— 这三个是**运行时按文件读取**的资源，Vercel 的打包器不会自动跟踪，不加就会在函数里缺文件。

---

## 🚀 部署

### 用 Vercel 部署

```bash
npm i -g vercel
vercel            # 按提示走；或直接在 Vercel 面板里 Import 本仓库
```

### 环境变量

在 Vercel 项目的 **Settings → Environment Variables** 里配置 PostgreSQL 连接（见 [`.env.example`](.env.example)）：

| 变量 | 说明 |
| --- | --- |
| `POSTGRES_HOST` / `POSTGRES_PORT` | 数据库地址与端口 |
| `POSTGRES_DATABASE` | 库名 |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` | 账号与密码 |
| `POSTGRES_PREFIX` | 表名前缀 |
| `POSTGRES_SSL` | 是否启用 SSL |

部署完成后，把地址填回博客的 `site-meta.ts`：

```ts
// valaxy/demo/yun/site-meta.ts
export const WALINE_SERVER_URL = 'https://<你的-project>.vercel.app'
```

---

## 🔧 定制

- **保存评论后做点事**：写 `index.cjs` 里的 `postSave(comment)` 钩子（比如通知、审核、同步）。
- **加插件**：往 `Application({ plugins: [...] })` 里塞 Waline 的插件。
- **管理后台**：Waline 自带，首个注册的用户成为管理员（见 [Waline 文档](https://waline.js.org/)）。

---

## 📄 许可

本仓库**未附带开源许可协议** —— 它是一份部署配置，评论系统的许可遵循 [Waline](https://github.com/walinejs/waline) 的 MIT。
