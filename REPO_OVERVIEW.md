# Uptime Kuma – Repository Overview

Tài liệu tóm tắt: tính năng, routes, API và cách chạy local (không dùng Docker Compose).

---

## 1. Tổng quan dự án

**Uptime Kuma** là công cụ self-hosted để giám sát uptime (HTTP, TCP, Ping, DNS, Docker, game server, database, v.v.), có giao diện web, status page, thông báo qua nhiều kênh (Telegram, Discord, email, webhook, …).

- **Backend:** Node.js (>= 20.4), Express, Socket.IO, Knex/RedBean (SQLite/MySQL/PostgreSQL/MariaDB).
- **Frontend:** Vue 3, Vite, Vue Router, Vue i18n.
- **Giao tiếp chính:** WebSocket (Socket.IO); một số API REST (push, badge, status page, metrics).

---

## 2. Các tính năng chính

### 2.1 Monitoring (Monitor types)

- **HTTP(s)** – GET/POST, keyword, JSON query, OAuth2, TLS, custom headers, basic auth.
- **TCP** – Kiểm tra port mở.
- **Ping** – ICMP (với tùy chọn numeric, count, timeout).
- **DNS** – Kiểm tra record (A, AAAA, CNAME, …), resolve type/server.
- **Websocket** – Kết nối WS, optional subprotocol.
- **Push** – Monitor nhận heartbeat từ bên ngoài qua API push.
- **Steam Game Server (GameDig)** – Game server query.
- **Docker** – Trạng thái container trên Docker host.
- **Database:** MySQL, PostgreSQL, MariaDB, MongoDB, MSSQL, Redis, RabbitMQ.
- **gRPC** – Health check gRPC (protobuf).
- **MQTT** – Publish/subscribe, WebSocket path.
- **SMTP** – Kiểm tra server mail.
- **SNMP** – SNMP check.
- **Kafka** – Producer check.
- **SIP OPTIONS** – SIP heartbeat.
- **Tailscale Ping** – Ping qua Tailscale.
- **Real Browser** – Kiểm tra bằng headless Chrome (screenshot, certificate).
- **System Service** – Trạng thái systemd service.
- **Manual** – Trạng thái set tay.
- **Group** – Nhóm monitor (parent/child), dùng cho status page.

### 2.2 Status Page

- Nhiều status page, mỗi page có slug, title, icon, theme.
- Map domain → status page (reverse proxy / domain mapping).
- RSS feed cho từng status page.
- Badge tổng thể (Up/Degraded/Down/Maintenance).
- API công khai: config, heartbeat, manifest, badge.

### 2.3 Thông báo (90+ kênh)

- Telegram, Discord, Slack, Microsoft Teams, Google Chat, Matrix, Mattermost, Rocket.Chat, …
- Email (SMTP, SendGrid, Brevo, Resend, …).
- SMS (Twilio, 46elks, SerwerSMS, …).
- Push: Pushover, Pushbullet, Ntfy, Gotify, Web Push, Bark, …
- Webhook, Apprise, PagerDuty, Opsgenie, SIGNL4, Splunk, …
- Nhiều provider khác (xem `server/notification-providers/` và `src/components/notifications/`).

### 2.4 Bảo mật & quản trị

- Đăng nhập (username/password), JWT, “login by token”.
- 2FA (TOTP).
- API Keys (tạo/xóa/enable/disable) – dùng cho Basic Auth (ví dụ `/metrics`).
- Có thể tắt auth (disableAuth) cho môi trường tin cậy.
- Đổi mật khẩu, reset password (CLI script).

### 2.5 Cấu hình & vận hành

- **Settings:** General (timezone, entry page, auth, Chrome path, NSCD, …), Appearance, Notifications, Tags, Monitor History, Reverse Proxy, Docker Hosts, Remote Browsers, Security, API Keys, Proxies, About.
- **Proxy:** HTTP proxy cho từng monitor (proxy list trong Settings).
- **Docker Host:** Thêm/xóa host Docker để monitor container.
- **Remote Browser:** Cấu hình browser từ xa cho Real Browser monitor.
- **Maintenance:** Khung bảo trì theo thời gian, gắn monitor/status page.
- **Cloudflared:** Tunnel (optional) qua Cloudflare.
- **Prometheus:** Endpoint `/metrics` (Basic Auth bằng user hoặc API key).
- **Database:** SQLite (mặc định), MySQL, PostgreSQL, MariaDB; có bước setup DB riêng.

### 2.6 Giao diện & trải nghiệm

- Dashboard: danh sách monitor, heartbeat bar, uptime %, ping chart.
- Chi tiết monitor: heartbeat history, ping chart, certificate, điều kiện (monitor conditions).
- Monitor conditions: biểu thức (AND/OR) dựa trên status, ping, keyword, …
- Tag cho monitor, filter theo tag.
- Đa ngôn ngữ (i18n), RTL.
- PWA (service worker).

---

## 3. Routes (Frontend – Vue Router)

Tất cả routes (trừ entry `/`) có **prefix `kuma-`** trong path, ví dụ `/dashboard` → `/kuma-dashboard`.

| Path | Component | Mô tả |
|------|-----------|--------|
| `/` | Entry | Entry page (redirect theo domain/entry page) |
| `/kuma-dashboard` | DashboardHome | Trang chủ dashboard |
| `/kuma-dashboard/:id` | Details | Chi tiết monitor |
| `/kuma-edit/:id` | EditMonitor | Sửa monitor |
| `/kuma-add` | EditMonitor | Thêm monitor |
| `/kuma-clone/:id` | EditMonitor | Clone monitor |
| `/kuma-list` | List | Danh sách monitor (list view) |
| `/kuma-settings` | Settings | Cài đặt (container) |
| `/kuma-settings/general` | General | Cài đặt chung |
| `/kuma-settings/appearance` | Appearance | Giao diện |
| `/kuma-settings/notifications` | Notifications | Thông báo |
| `/kuma-settings/reverse-proxy` | ReverseProxy | Reverse proxy |
| `/kuma-settings/tags` | Tags | Tags |
| `/kuma-settings/monitor-history` | MonitorHistory | Lịch sử monitor |
| `/kuma-settings/docker-hosts` | Docker | Docker hosts |
| `/kuma-settings/remote-browsers` | RemoteBrowsers | Remote browsers |
| `/kuma-settings/security` | Security | Bảo mật (2FA, password) |
| `/kuma-settings/api-keys` | APIKeys | API Keys |
| `/kuma-settings/proxies` | Proxies | Proxies |
| `/kuma-settings/about` | About | Giới thiệu / phiên bản |
| `/kuma-manage-status-page` | ManageStatusPage | Quản lý status page |
| `/kuma-add-status-page` | AddStatusPage | Thêm status page |
| `/kuma-maintenance` | ManageMaintenance | Quản lý maintenance |
| `/kuma-add-maintenance` | EditMaintenance | Thêm maintenance |
| `/kuma-maintenance/edit/:id` | EditMaintenance | Sửa maintenance |
| `/kuma-maintenance/clone/:id` | EditMaintenance | Clone maintenance |
| `/kuma-setup` | Setup | Setup lần đầu (tạo user) |
| `/kuma-setup-database` | SetupDatabase | Chọn DB (SQLite/MySQL/PostgreSQL/MariaDB) |
| `/kuma-status-page` | StatusPage | Status page (slug default) |
| `/kuma-status` | StatusPage | Status page (slug default) |
| `/kuma-status/:slug` | StatusPage | Status page theo slug |
| `*` | NotFound | 404 |

---

## 4. HTTP Routes (Backend – Express)

### 4.1 Server gốc (`server.js`)

| Method | Path | Auth | Mô tả |
|--------|------|------|--------|
| GET | `/` | — | Entry: redirect theo domain / entry page / status page mapping |
| GET | `/setup-database-info` | — | Trạng thái setup DB (runningSetup, needSetup) |
| GET | `/robots.txt` | — | Robots.txt (Disallow nếu không cho index) |
| GET | `/metrics` | apiAuth (Basic) | Prometheus metrics |
| GET | `/.well-known/change-password` | — | Redirect tới wiki reset password |
| GET | `*` | — | Serve SPA (indexHTML) hoặc 404 cho `/upload/` |

Static:

- `/` → `dist/` (express-static-gzip)
- `/upload` → `data/upload`

Dev-only:

- POST `/test-webhook`, POST `/test-x-www-form-urlencoded`
- GET `/_e2e/take-sqlite-snapshot`, `/_e2e/restore-sqlite-snapshot`

### 4.2 API Router (`server/routers/api-router.js`)

| Method | Path | Auth | Mô tả |
|--------|------|------|--------|
| GET | `/api/entry-page` | — | Entry page / status page mapping (type, entryPage hoặc statusPageSlug) |
| ALL | `/api/push/:pushToken` | — | **Push heartbeat:** nhận status (up/down), msg, ping, trigger notification |
| GET | `/api/badge/:id/status` | — | Badge SVG trạng thái (Up/Down/Pending/Maintenance) |
| GET | `/api/badge/:id/uptime/:duration?` | — | Badge uptime % (duration: 24h, 720h, …) |
| GET | `/api/badge/:id/ping/:duration?` | — | Badge avg ping |
| GET | `/api/badge/:id/avg-response/:duration?` | — | Badge avg response time |
| GET | `/api/badge/:id/cert-exp` | — | Badge SSL cert còn lại (days/date) |
| GET | `/api/badge/:id/response` | — | Badge response time mới nhất |

Query params cho badge: `label`, `style`, màu (`upColor`, `downColor`, …), `value` (demo).

### 4.3 Status Page Router (`server/routers/status-page-router.js`)

| Method | Path | Auth | Mô tả |
|--------|------|------|--------|
| GET | `/status/:slug` | — | HTML status page |
| GET | `/status/:slug/rss` | — | RSS feed |
| GET | `/status` | — | Status page slug default |
| GET | `/status-page` | — | Status page slug default |
| GET | `/api/status-page/:slug` | — | Config + incidents + monitors (JSON) |
| GET | `/api/status-page/heartbeat/:slug` | — | Heartbeat + uptime 24h cho polling |
| GET | `/api/status-page/:slug/manifest.json` | — | PWA manifest |
| GET | `/api/status-page/:slug/badge` | — | Badge tổng (Up/Degraded/Down/Maintenance) |

---

## 5. API quan trọng (Socket.IO)

Giao tiếp chính của dashboard là **Socket.IO**. Client kết nối, gửi event, server reply qua callback hoặc emit.

### 5.1 Auth & Setup (public)

- `loginByToken` – Đăng nhập bằng JWT.
- `login` – Đăng nhập username/password (trả về JWT nếu không 2FA).
- `logout`
- `prepare2FA`, `save2FA`, `disable2FA`, `verifyToken`, `twoFAStatus`
- `needSetup` – Có cần setup không.
- `setup` – Tạo user đầu tiên.

### 5.2 Monitor (auth)

- `add` – Thêm monitor.
- `editMonitor` – Sửa monitor.
- `getMonitorList` – Danh sách monitor.
- `getMonitor` – Chi tiết 1 monitor.
- `checkMointor` – Hỗ trợ domain expiry (partial URL).
- `getMonitorBeats` – Heartbeat theo period.
- `resumeMonitor`, `pauseMonitor` – Bật/tắt monitor.
- `deleteMonitor` – Xóa (có thể xóa/unlink children nếu là group).
- `clearEvents`, `clearHeartbeats`, `clearStatistics` – Xóa events/heartbeat/stats.

### 5.3 Tags (auth)

- `getTags`, `addTag`, `editTag`, `deleteTag`
- `addMonitorTag`, `editMonitorTag`, `deleteMonitorTag`

### 5.4 Heartbeat / important (auth)

- `monitorImportantHeartbeatListCount` – Số heartbeat “important”.
- `monitorImportantHeartbeatListPaged` – Danh sách important heartbeat (offset, count).

### 5.5 User & Settings (auth)

- `changePassword`
- `getSettings` – Settings nhóm “general”.
- `setSettings` – Lưu general (entry page, timezone, auth, Chrome, …).

### 5.6 Notifications (auth)

- `addNotification`, `deleteNotification`
- `testNotification`
- `checkApprise`
- `getWebpushVapidPublicKey`

### 5.7 Status Page (auth – status-page-socket-handler)

- `postIncident`, `unpinIncident`
- `getStatusPage`, `saveStatusPage`
- `addStatusPage`, `deleteStatusPage`

### 5.8 Maintenance (auth – maintenance-socket-handler)

- `addMaintenance`, `editMaintenance`, `deleteMaintenance`
- `addMonitorMaintenance`, `addMaintenanceStatusPage`
- `getMaintenance`, `getMaintenanceList`, `getMonitorMaintenance`, `getMaintenanceStatusPage`
- `pauseMaintenance`, `resumeMaintenance`

### 5.9 Proxy, Docker, Remote Browser (auth)

- Proxy: `addProxy`, `deleteProxy`
- Docker: `addDockerHost`, `deleteDockerHost`, `testDockerHost`
- Remote Browser: `addRemoteBrowser`, `deleteRemoteBrowser`, `testRemoteBrowser`

### 5.10 API Key (auth – api-key-socket-handler)

- `addAPIKey`, `getAPIKeyList`, `deleteAPIKey`, `disableAPIKey`, `enableAPIKey`

### 5.11 Database, Cloudflared, General, Chart (auth)

- Database: `getDatabaseSize`, `shrinkDatabase`
- Cloudflared: `cloudflared_join`, `cloudflared_leave`, `cloudflared_start`, `cloudflared_stop`, `cloudflared_removeToken`
- General: `initServerTimezone`, `getGameList`, `testChrome`, `getPushExample`, `disconnectOtherSocketClients`
- Chart: `getMonitorChartData` – Dữ liệu chart theo period.

### 5.12 Server → Client (emit)

- `setup`, `loginRequired`, `autoLogin`
- `heartbeat` – Heartbeat mới.
- `monitorList`, `updateMonitor`, `deleteMonitor`
- `importantHeartbeatList`, `heartbeatList`, `stats`, …
- Các list: notificationList, proxyList, dockerHostList, apiKeyList, remoteBrowserList, monitorTypeList, maintenanceList, statusPageList, info, …

---

## 6. Setup & chạy local (không Docker Compose)

### 6.1 Yêu cầu

- **Node.js** >= 20.4 (khuyến nghị dùng `nvm use` theo `.nvmrc` nếu có).
- **Git**, **npm** (hoặc pnpm/yarn nếu project hỗ trợ).

### 6.2 Cài đặt

```bash
# Clone
git clone https://github.com/louislam/uptime-kuma.git
cd uptime-kuma

# Node version (nếu dùng nvm)
nvm use

# Cài dependency
npm ci
```

### 6.3 Chạy production (một process, dùng build sẵn)

Cần build frontend trước, sau đó chạy server:

```bash
npm run build
npm run start-server
# hoặc
node server/server.js
```

Mặc định: **http://0.0.0.0:3001** (hoặc `http://localhost:3001`). Port/host có thể đổi bằng:

- `--port=3002` / `--host=127.0.0.1`
- Hoặc env: `UPTIME_KUMA_PORT`, `UPTIME_KUMA_HOST`, `PORT`, `HOST`

Dữ liệu (DB, upload) nằm trong thư mục **data/** (tạo tự động nếu chưa có).

### 6.4 Chạy development (frontend + backend)

Frontend (Vite) chạy port **3000**, backend (Express) port **3001**. Script `dev` đợi backend lên rồi mới chạy frontend:

```bash
npm run dev
```

- Backend: `npm run start-server-dev` → `http://localhost:3001`
- Frontend: `npm run start-frontend-dev` → `http://localhost:3000` (Vite, hot reload)

Truy cập app qua **http://localhost:3000**; frontend proxy / gọi API/WS tới backend 3001 (xem `config/vite.config.js` và `server/config.js`).

### 6.5 Biến môi trường (.env)

Tất cả biến đều **tùy chọn**; app có sẵn giá trị mặc định. Server đọc `.env` qua `dotenv` khi khởi động.

- **Tạo file .env:** copy từ `env.example`: `cp env.example .env` rồi sửa.
- **Các nhóm biến chính:** server (host, port), base path (`UPTIME_KUMA_BASE_PATH`), data dir, TLS, WebSocket/security, Cloudflare tunnel, container, Chrome, notification proxy, logging, database (type, host, user, password, socket, SSL, pool). Chi tiết từng biến xem trong `env.example`.

### 6.6 Chạy nền bằng PM2 (không Docker)

```bash
npm run build
pm2 start server/server.js --name uptime-kuma
pm2 save
pm2 startup   # optional: khởi động cùng OS
```

### 6.7 Base path (ví dụ: m.sankakuapi.com/uptime)

Khi chạy app dưới một path cố định (ví dụ **https://m.sankakuapi.com/uptime**) qua nginx, cần set **cùng base path** cho build và runtime:

1. **Build frontend** với base path `/uptime`:
   ```bash
   UPTIME_KUMA_BASE_PATH=/uptime npm run build
   ```

2. **Chạy server** với cùng biến:
   ```bash
   UPTIME_KUMA_BASE_PATH=/uptime node server/server.js
   ```
   Hoặc export trước: `export UPTIME_KUMA_BASE_PATH=/uptime`

3. **Nginx**: proxy toàn bộ path `/uptime` (và `/uptime/`) về backend **không strip prefix** (backend nhận path đầy đủ `/uptime/...`), ví dụ:
   ```nginx
   location /uptime {
       proxy_pass http://127.0.0.1:3001;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
   }
   ```

Khi đó: routes, redirect và API đều dùng prefix `/uptime` (ví dụ `/uptime/kuma-dashboard`, `/uptime/api/...`).

---

## 7. Tóm tắt file quan trọng

| Vai trò | Path |
|--------|------|
| Entry server | `server/server.js` |
| Cấu hình port/host/SSL | `server/config.js` |
| API REST (push, badge, entry-page) | `server/routers/api-router.js` |
| Status page REST + HTML/RSS | `server/routers/status-page-router.js` |
| Auth (login, apiAuth) | `server/auth.js` |
| Socket handlers | `server/socket-handlers/*.js` |
| Monitor types | `server/monitor-types/*.js` |
| Notification providers | `server/notification-providers/*.js` |
| Models | `server/model/*.js` |
| Vue router | `src/router.js` |
| Vite config | `config/vite.config.js` |
| Scripts | `package.json` → `dev`, `start-server`, `build` |

---

*Tài liệu được sinh từ cấu trúc repo tại thời điểm đọc; có thể cần cập nhật khi code thay đổi.*
