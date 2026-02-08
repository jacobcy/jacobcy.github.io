# CLAUDE.md - iWangPo Blog 项目配置

## 项目基本信息
- **名称**: iWangPo Blog
- **网址**: https://iwangpo.com
- **平台**: Hexo 8.27.0 + NexT Gemini 主题
- **部署**: 静态站点生成 + Nginx 反向代理

## 技术栈
- Hexo 8.27.0
- NexT Gemini 主题
- Node.js
- Nginx

## 重要路径
```
/home/iwangpo/app/hexo/
├── source/_posts/          # 文章源文件 (.md)
├── source/about/index.md   # 关于页面
├── _config.yml            # Hexo 主配置
├── _config.next.yml       # NexT 主题配置
└── public/                # 生成的静态站点
```

## 常用命令
```bash
cd /home/iwangpo/app/hexo

# 开发
npx hexo server            # 本地预览 (localhost:4000)
npx hexo generate          # 生成静态文件
npx hexo clean             # 清理缓存

# 部署 (如果需要)
npx hexo deploy            # 部署到配置的目标
```

## 内容规范

### Front-matter 模板
```yaml
---
title: 文章标题
date: 2025-01-01 10:00:00
tags:
  - 标签1
  - 标签2
categories:
  - 分类名
---
```

### 内容分类
- `tech` - 技术/AI
- `quant` - 量化/投资
- `life` - 生活/咖啡
- `photo` - 摄影/电影

## SEO 配置状态
- robots.txt: ✅ 已配置
- sitemap.xml: ✅ 自动生成
- atom.xml: ✅ RSS feed
- Open Graph: ✅ 已启用
- JSON-LD: ✅ 已启用

## NexT 主题关键配置
文件: `_config.next.yml`

```yaml
# 菜单
menu:
  home: / || fa fa-home
  archives: /archives/ || fa fa-archive
  tags: /tags/ || fa fa-tags
  about: /about/ || fa fa-user

# SEO
seo: true
index_with_subtitle: true

# 搜索
local_search:
  enable: true

# 社交链接 (如需要)
social:
  GitHub: https://github.com/username || fab fa-github
```

## 快速参考
- 当前文章数: 23 篇
- 内容时间范围: 2025-02 至 2026-01
- 博主身份: 程序员 × 咖啡信徒 × 效率控 × 半吊子摄影师

---
*用途: 供 AI Agent 快速了解项目配置*
