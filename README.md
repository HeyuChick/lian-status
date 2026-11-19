# lian-status

涟（Lian）服务器监控探针前端 —— [ServerStatus-Rust](https://github.com/zdz/ServerStatus-Rust) 自定义单文件主题，视觉风格延续自个人主页 [HeyuChick/lian-site](https://github.com/HeyuChick/lian-site)。

- 在线演示：https://status.heyuchick.com

## 特性

- **涟（Lian）视觉体系**：`#0a0a0a` / `#111113` / `#1f1f22` 极简深色与 `1px` 细线布局，搭配 `JetBrains Mono`、`Noto Sans SC` / `Noto Sans JP` 及 `tabular-nums` 等宽数字。
- **动态氛围与交互**：沿用 `<a-waves>` WebGL2 Perlin 噪声流线背景（支持 Canvas2D 降级及 `prefers-reduced-motion`）、`<lian-cursor>` 自定义阻尼光标与 `#goo` 点击黏液涟漪；主题强调色 `--goo-color` 随集群在线健康度与实时负载平滑切换。
- **多语言同步**：内置 `中 / EN / 日` 切换，并与 `lian-site` 共用 `localStorage` 中的 `lian-lang` 偏好设置。
- **高效监控体验**：
  - 4 栏集群汇总条：在线节点数、实时总带宽、本月总流量、集群平均负载与内存。
  - 状态 / 地区筛选、多维度排序（权重 / CPU / 内存 / 网速）以及 **列表（LIST）/ 卡片（GRID）** 双视图切换。
  - 基于节点 `name` 的 2 秒原地增量刷新，保留展开状态不闪烁。
  - 节点详情抽屉内嵌 **近 60 次采样的 CPU / 内存 / 网速 SVG 折线图**、物理内存/Swap、磁盘、本月与累计流量、TCP/UDP/进程数、中国三网（电信 / 联通 / 移动）延迟与丢包详情、`ndd` 到期倒计时及心跳延迟。

## 部署示例（Nginx）

直接将本仓库根目录的 `index.html` 部署到静态目录（例如 `/var/www/serverstatus/index.html`），并由 Nginx 前置代理到 `stat_server`：

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name status.example.com;

    location = / {
        root /var/www/serverstatus;
        try_files /index.html =404;
        add_header Cache-Control "no-cache" always;
    }

    location = /index.html {
        root /var/www/serverstatus;
        add_header Cache-Control "no-cache" always;
    }

    # 保留原版前端入口（可选）
    location = /legacy {
        rewrite ^/legacy$ / break;
        proxy_pass http://127.0.0.1:8088;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 透传 /json/stats.json、/detail、/map 及静态资源
    location / {
        proxy_pass http://127.0.0.1:8088;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
