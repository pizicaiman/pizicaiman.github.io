# 基于Next.js开发个人博客 设计及关键代码举例

## 1. 博客平台基本设计要素

- 支持文章发布、编辑、删除
- 文章标签分类、按时间归档
- 支持 Markdown 写作
- 评论与用户互动（可选）
- SEO 优化、RSS 输出
- 文章检索（全文或标题搜索）
- 皮肤定制与响应式布局

## 2. 技术选型

- 前端渲染框架：Next.js (React基础，支持SSG/SSR)
- 数据源：本地 Markdown 文件 / Headless CMS (如Notion, Strapi, Sanity) / 直连DB（简单项目建议用MarkDown文件）
- 样式与UI：Tailwind CSS/Ant Design/Chakra-UI任选
- 部署：Vercel/Netlify/自托管服务器
- 评论功能：Gitalk/Disqus/微服务后端

## 3. Next.js 关键文件结构设计

```
.
├── pages/
│   ├── index.tsx           # 主页，文章列表
│   ├── [slug].tsx          # 文章详情页（动态路由）
│   ├── tag/[tag].tsx       # 标签归档
│   └── api/                # API 路径（如评论提交）
├── posts/                  # Markdown文章内容
│   ├── welcome-to-blog.md
│   └── ...
├── components/
│   ├── Layout.tsx
│   ├── PostCard.tsx
│   └── MarkdownRender.tsx
├── lib/
│   ├── posts.ts            # 文档读取与处理
├── public/
│   └── logo.png
├── styles/
└── next.config.js
```

## 4. Markdown 文章读取与 SSG 实现（简化示例）

**lib/posts.ts**

```typescript
import fs from 'fs'
import path from 'path'
import matter from 'gray-matter'

const postsDirectory = path.join(process.cwd(), 'posts')

export function getAllPostSlugs() {
  return fs.readdirSync(postsDirectory)
    .filter(file => file.endsWith('.md'))
    .map(file => ({
      params: { slug: file.replace(/\.md$/, '') }
    }))
}

export function getPostData(slug: string) {
  const fullPath = path.join(postsDirectory, `${slug}.md`)
  const fileContents = fs.readFileSync(fullPath, 'utf8')
  const { data, content } = matter(fileContents)
  return {
    slug,
    ...(data as { title: string; date: string; tags?: string[] }),
    content,
  }
}

export function getSortedPostsData() {
  const fileNames = fs.readdirSync(postsDirectory)
  const allPostsData = fileNames.map(fileName => {
    const slug = fileName.replace(/\.md$/, '')
    return getPostData(slug)
  })
  return allPostsData.sort((a, b) =>
    a.date < b.date ? 1 : -1
  )
}
```

---

## 5. 博客主页实现（简化版）

**pages/index.tsx**

```typescript
import { GetStaticProps } from 'next'
import Link from 'next/link'
import { getSortedPostsData } from '../lib/posts'

export default function Home({ allPosts }) {
  return (
    <div>
      <h1>我的Next.js博客</h1>
      <ul>
        {allPosts.map(({ slug, title, date }) => (
          <li key={slug}>
            <Link href={`/${slug}`}>
              {title} ({date})
            </Link>
          </li>
        ))}
      </ul>
    </div>
  )
}

export const getStaticProps: GetStaticProps = async () => {
  const allPosts = getSortedPostsData()
  return {
    props: { allPosts }
  }
}
```

---

## 6. 文章详情页动态路由实现

**pages/[slug].tsx**

```typescript
import { GetStaticPaths, GetStaticProps } from 'next'
import { getAllPostSlugs, getPostData } from '../lib/posts'
import ReactMarkdown from 'react-markdown'

export default function Post({ postData }) {
  return (
    <article>
      <h1>{postData.title}</h1>
      <div>{postData.date}</div>
      <div>
        <ReactMarkdown>{postData.content}</ReactMarkdown>
      </div>
    </article>
  )
}

export const getStaticPaths: GetStaticPaths = async () => {
  const paths = getAllPostSlugs()
  return { paths, fallback: false }
}

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const postData = getPostData(params!.slug as string)
  return {
    props: { postData }
  }
}
```

---

## 7. 评论/皮肤/分类等后续拓展点

- 评论可用 Vercel Serverless API 或第三方平台
- 可增加主题切换、站点基本SEO信息、RSS输出等
- 增加 `tag` 页面按标签归档
- 添加全文搜索功能可用 fuse.js 前端实现
- 可用 `getServerSideProps` 支持动态内容或登录校验

---

**总结：**  
Next.js 博客本质即 “内容读取（可用md文件或CMS）、页面静态或动态生成、样式优化和功能扩展”，非常适合个人技术博客网站构建与运维。

