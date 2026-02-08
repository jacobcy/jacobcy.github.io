# Nginx 站点配置记录

## 概述
- **主配置目录**: `/etc/nginx/sites-enabled/`
- **SSL通用配置**: `/etc/nginx/conf.d/ssl_common.conf`
- **优化日期**: 2026年2月6日

---

## 1. bghunt.cn 站点

### 主域名
| 域名 | 类型 | 后端 | 用途 |
|------|------|------|------|
| bghunt.cn | 静态站点 | `/home/iwangpo/scripts/nginx/bghunt.cn` | 主网站 |
| www.bghunt.cn | 静态站点 | `/home/iwangpo/scripts/nginx/bghunt.cn` | 主网站WWW |

### 子域名代理
| 子域名 | 端口 | 用途 |
|--------|------|------|
| api.bghunt.cn | 7086 | API服务 |
| silly.bghunt.cn | 7089 | Silly服务 |
| dsm.bghunt.cn | 7070 | DSM服务 |
| file.bghunt.cn | 7070 | 文件服务 |
| photo.bghunt.cn | 7070 | 照片服务 |

### 配置特点
- 使用 `map $host $backend_port` 统一管理代理端口
- 静态资源缓存1年
- 启用 HTTP/2

---

## 2. iwangpo.com 站点

### 主域名
| 域名 | 类型 | 后端 | 用途 |
|------|------|------|------|
| iwangpo.com | 静态站点 | `/home/iwangpo/app/hexo/public` | Hexo博客 |
| www.iwangpo.com | 静态站点 | `/home/iwangpo/app/hexo/public` | 博客WWW |

### 配置特点
- Hexo静态博客托管
- 图片缓存1年，CSS/JS缓存7天，HTML缓存1小时
- 使用 map 智能重定向 HTTP 到 HTTPS

---

## 3. 已移除的端点 (2026-02-06清理)

### bghunt.cn (后端未运行)
- ~~dev.bghunt.cn~~ (7087端口服务异常)
- ~~docs.bghunt.cn~~ (7088端口服务异常)
- ~~mem0.bghunt.cn~~ (7092端口未监听)
- ~~mem.bghunt.cn~~ (7093端口未监听)

### iwangpo.com (服务不存在)
- ~~admin.iwangpo.com~~ (静态目录不存在)
- ~~testadmin.iwangpo.com~~ (9000端口未监听)
- ~~testapi.iwangpo.com~~ (8000端口未监听)
- ~~wx.iwangpo.com~~ (6060端口未监听)

---

## 4. 性能优化配置

### SSL/TLS
- 仅支持 TLSv1.2 和 TLSv1.3
- 启用 HTTP/2
- SSL会话缓存 10m

### 缓存策略
```nginx
# 图片/字体/ICO - 1年
location ~* \.(jpg|jpeg|png|gif|ico|svg|woff|woff2|ttf|eot)$ {
    expires 1y;
}

# CSS/JS - 7天
location ~* \.(css|js)$ {
    expires 7d;
}

# HTML - 1小时
location ~* \.(html|htm)$ {
    expires 1h;
}
```

### 安全头
- X-Frame-Options: SAMEORIGIN
- X-Content-Type-Options: nosniff
- X-XSS-Protection: 1; mode=block
- Referrer-Policy: strict-origin-when-cross-origin

---

## 5. 常用命令

```bash
# 测试配置
sudo nginx -t

# 重载配置
sudo systemctl reload nginx

# 检查端口监听
sudo ss -tlnp | grep -E ':7070|:7086|:7089'

# 测试端点
for domain in bghunt.cn api.bghunt.cn iwangpo.com; do
    curl -s -o /dev/null -w "$domain: %{http_code}\n" https://$domain
done
```

---

## 6. 文件位置

```
/etc/nginx/
├── nginx.conf                    # 主配置
├── sites-enabled/
│   ├── bghunt.cn.conf           # bghunt站点配置
│   └── iwangpo.com.conf         # iwangpo站点配置
├── conf.d/
│   └── ssl_common.conf          # SSL通用配置
└── dhparam.pem                  # DH参数
```

---
*记录时间: 2026-02-06*
*记录内容: 当前运行的所有nginx站点及配置*
