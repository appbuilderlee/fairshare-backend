# FairShare Backend - Cloudflare Workers

完整的 Cloudflare Workers 後端，為 FairShare Bill Splitter 提供安全的 API 服務。

## 功能

- ✅ **AI API 代理** - 隱藏 Gemini API Key，前端只呼叫 `/api/ai/*`
- ✅ **登入註冊** - 使用 Turnstile 防機器人 + JWT 認證
- ✅ **帳單 CRUD** - 支援多人拆帳，儲存在 Cloudflare D1
- ✅ **CORS 設定** - 前端跨域請求支援

## 專案結構

```
fairshare-backend/
├── src/
│   ├── index.ts              # 主路由
│   ├── routes/
│   │   ├── auth.ts           # 登入/註冊
│   │   ├── ai.ts             # AI API 代理
│   │   └── bills.ts          # 帳單 CRUD
│   ├── middleware/
│   │   ├── auth.ts           # JWT 驗證
│   │   └── cors.ts           # CORS 處理
│   ├── services/
│   │   ├── turnstile.ts      # Turnstile 驗證
│   │   ├── jwt.ts            # JWT 產生/驗證
│   │   └── gemini.ts         # Gemini AI 呼叫
│   └── db/
│       └── schema.sql        # D1 資料庫結構
├── wrangler.toml             # Cloudflare 設定
├── package.json
└── tsconfig.json
```

## 安裝與部署

### 1. 安裝依賴

```bash
npm install
```

### 2. 建立 D1 資料庫

```bash
# 建立資料庫
npx wrangler d1 create fairshare

# 記下 database_id，更新到 wrangler.toml
# 執行 schema
npx wrangler d1 execute fairshare --file=src/db/schema.sql
```

### 3. 設定 Secrets

```bash
# Turnstile Secret Key (從 Cloudflare Dashboard 取得)
npx wrangler secret put TURNSTILE_SECRET_KEY

# Gemini API Key (從 Google AI Studio 取得)
npx wrangler secret put GEMINI_API_KEY

# JWT Secret (隨機字串)
npx wrangler secret put JWT_SECRET
```

### 4. 本地開發

```bash
npm run dev
```

### 5. 部署到 Cloudflare

```bash
npm run deploy
```

部署後會得到 Worker URL，例如：`https://fairshare-backend.YOUR-SUBDOMAIN.workers.dev`

## API 端點

### 認證 API

**註冊**
```
POST /api/auth/register
Body: {
  "email": "user@example.com",
  "password": "password123",
  "name": "使用者",
  "turnstileToken": "..."
}
```

**登入**
```
POST /api/auth/login
Body: {
  "email": "user@example.com",
  "password": "password123",
  "turnstileToken": "..."
}
```

### AI API

**拆帳分析**
```
POST /api/ai/analyze
Headers: Authorization: Bearer <JWT>
Body: {
  "totalAmount": 1000,
  "participants": ["Alice", "Bob", "Charlie"],
  "payments": [
    {"name": "Alice", "paid": 1000}
  ]
}
```

### 帳單 API

**取得所有帳單**
```
GET /api/bills
Headers: Authorization: Bearer <JWT>
```

**建立帳單**
```
POST /api/bills
Headers: Authorization: Bearer <JWT>
Body: {
  "title": "晚餐",
  "totalAmount": 300,
  "currency": "TWD",
  "items": [...]
}
```

## 整合到前端

更新你的前端 `.env`：

```env
VITE_API_URL=https://fairshare-backend.YOUR-SUBDOMAIN.workers.dev
VITE_TURNSTILE_SITE_KEY=your-turnstile-site-key
```

前端呼叫範例：

```typescript
// services/api.ts
const API_URL = import.meta.env.VITE_API_URL;

export async function login(email: string, password: string, turnstileToken: string) {
  const response = await fetch(`${API_URL}/api/auth/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password, turnstileToken })
  });
  const data = await response.json();
  localStorage.setItem('token', data.token);
  return data;
}

export async function getBills() {
  const token = localStorage.getItem('token');
  const response = await fetch(`${API_URL}/api/bills`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  return response.json();
}
```

## 完整程式碼檔案

完整的 TypeScript 原始碼請見以下檔案：

- `src/index.ts` - 主路由
- `src/routes/*` - API 路由
- `src/middleware/*` - 中間件
- `src/services/*` - 服務層
- `src/db/schema.sql` - 資料庫結構

## License

MIT
