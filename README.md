# Blog

个人博客，当前直接部署在 CloudFlare Pages 上，另外还通过 GitHub Actions 部署到 CloudFlare Workers 上

## CloudFlare Pages

丝滑流畅，直接在 CloudFlare 上链接博客的 repo，配置构建命令 `npm run build`，配置输出目录为 `./public` 即可

此部分配置对应着 Hexo 的构建行为

## 通过 GitHub Actions 部署到 CloudFlare Workers

[![Deploy](https://github.com/weremexii/blog/actions/workflows/Deploy.yaml/badge.svg)](https://github.com/weremexii/blog/actions)

部署到 Workers 上的配置文件参见 `.github/workflows/Deploy.yaml`，内容主要参考

[将 Hexo 部署到 Cloudflare Workers Site 上的趟坑记录 | Sukka's Blog](https://blog.skk.moe/post/deploy-blog-to-cf-workers-site/)
