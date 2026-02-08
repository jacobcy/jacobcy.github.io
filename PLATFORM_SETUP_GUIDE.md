# 个人品牌矩阵 - 平台配置指南

## 已完成的更新

### ✅ 1. bghunt.cn (产品展示页)
**位置**: `/home/iwangpo/scripts/nginx/bghunt.cn/index.html`

**更新内容**:
- 添加顶部导航栏（项目、关于、博客、GitHub 链接）
- Hero 区域添加人设标签和社交图标（GitHub、博客、X、Telegram）
- About 区域更新为真实社交链接

---

### ✅ 2. iwangpo.com (博客)
**位置**: `/home/iwangpo/hexo/source/about/index.md`

**更新内容**:
- 关于页面添加完整的「全网同号」板块
- 包含所有 5 个平台链接

**位置**: `/home/iwangpo/hexo/_config.next.yml`

**更新内容**:
- 配置 NexT 主题社交链接（GitHub、Twitter、Telegram）

**下一步**: 运行 `npx hexo generate` 重新生成站点

---

### ✅ 3. GitHub Profile
**位置**: `/home/iwangpo/hexo/github-profile-readme.md`

**使用方式**:
1. 创建仓库 `github.com/jacobcy/jacobcy`（与用户名相同的仓库）
2. 上传上述 README.md 文件
3. 效果：访问 github.com/jacobcy 时显示自定义 Profile

---

## 需要手动配置的平台

### 🐦 4. X/Twitter Bio 配置

**建议 Bio 文案** (160字符以内):
```
程序员 × 咖啡信徒 × 效率控
☕ 代码改变生活 | 咖啡治愈灵魂
📝 iwangpo.com | ✈️ t.me/shark_fisher_channel
🚀 bghunt.cn
```

**建议置顶推文**:
```
欢迎来到我的数字花园 🌱

这里记录我的技术探索、咖啡时光和生活思考。

📝 深度文章 → iwangpo.com
🚀 项目展示 → bghunt.cn  
💻 代码仓库 → github.com/jacobcy
✈️ 独家内容 → t.me/shark_fisher_channel

一起交流，共同成长 ☕
```

**Website 字段**: https://iwangpo.com

---

### ✈️ 5. Telegram 频道描述

**频道名称**: Shark Fisher Channel（或你的偏好）

**建议描述文案**:
```
🦈 程序员的数字游民生活

分享技术干货、投资思考、咖啡笔记和生活碎片。

📖 深度文章: iwangpo.com
🐦 日常动态: x.com/Lullaby_shark
💻 开源项目: github.com/jacobcy

欢迎订阅，一起探索 ☕
```

---

## 流量闭环策略

### 内容分发 SOP

**每周发布节奏**:
```
周一: Telegram 独家内容（深度思考/资源分享）
周三: 博客长文发布 → X 发摘要/金句
周五: X 技术观察/生活分享
周末: Telegram 推送本周博客精华
```

**跨平台引流话术**:

| 场景 | 引流方式 |
|------|----------|
| X 发技术观点 | "详细实现 → iwangpo.com" |
| 博客文章底部 | "更多实时思考 → 关注我的 X" |
| Telegram 推送 | "原文链接 + 我的独家点评" |
| GitHub 项目 | "实现详解 → iwangpo.com/blog/xxx" |
| bghunt.cn 项目 | "技术博客详解 → 查看文章" |

### 人设一致性

**统一标签**: 程序员 × 咖啡信徒 × 效率控 × 半吊子摄影师

**统一口号**: "代码可以改变世界，但生活不能被代码替代"

**统一视觉**:
- 所有平台使用相同头像
- 配色风格保持一致（深色 + 蓝色强调）
- 简介风格统一（中文为主，英文为辅）

---

## 下一步行动清单

- [ ] 运行 `npx hexo generate` 重新生成博客
- [ ] 上传 GitHub Profile README 到 jacobcy/jacobcy 仓库
- [ ] 更新 X Bio 和置顶推文
- [ ] 更新 Telegram 频道描述
- [ ] 测试所有链接是否正常跳转
- [ ] 制定第一周内容发布计划

---

## 平台矩阵图

```
                    ┌─────────────────┐
                    │   iwangpo.com   │
                    │   (内容中枢)    │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  bghunt.cn    │   │ github.com/   │   │ x.com/        │
│  (产品展示)   │   │ jacobcy       │   │ Lullaby_shark │
│               │   │ (技术背书)    │   │ (日常发声)    │
└───────────────┘   └───────────────┘   └───────┬───────┘
                                                │
                                                ▼
                                       ┌───────────────┐
                                       │ t.me/shark_   │
                                       │ fisher_channel│
                                       │ (私域沉淀)    │
                                       └───────────────┘
```

所有平台已相互链接，形成完整闭环！
