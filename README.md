# ZHUO MARKET — Backend

Backend starter for the current ZhuoMarket HTML UI. It provides authentication, products/categories, wishlist, notifications, orders/checkout, downloads, support tickets, admin API, and payment/webhook endpoints.

## 1. Install

```bash
npm install
cp .env.example .env
```

Edit `.env` before launching. **Never put secret keys in the HTML/APK.**

## 2. Seed the database

```bash
npm run seed
```

This creates the SQLite database and the admin account from `ADMIN_EMAIL` / `ADMIN_PASSWORD`.

## 3. Start

Development:

```bash
npm run dev
```

Production:

```bash
npm start
```

Health check: `GET /api/health`

## 4. API base URL for the HTML

The frontend should use:

```js
const API_BASE_URL = 'http://localhost:8080/api';
```

For deployment replace it with your HTTPS backend URL, e.g. `https://api.yourdomain.com/api`.

## 5. Endpoints used by the app

### Auth
- `POST /api/auth/register` `{email,password,name}`
- `POST /api/auth/login` `{email,password}`
- `GET /api/me` Bearer token

### Store
- `GET /api/categories`
- `GET /api/products?cat=freefire&q=...`
- `GET /api/products/:id`
- `GET /api/wishlist`
- `POST /api/wishlist/:productId`

### User data
- `GET /api/notifications`
- `POST /api/notifications/read`
- `GET /api/orders`
- `GET /api/downloads`
- `POST /api/support/tickets`
- `GET /api/support/tickets`

### Checkout / payments
- `POST /api/checkout` `{productId,qty,method}`
- `POST /api/payments/:provider/create` `{orderId}`
- `POST /api/payments/webhook/:provider`

The payment endpoints intentionally do not pretend that a payment is successful. Live MonCash/NatCash/PayPal/Stripe credentials and provider-specific webhook/signature verification must be configured before accepting real payments.

### Admin (backend only)
Requires an admin JWT:
- `GET /api/admin/stats`
- `POST /api/admin/products`
- `PATCH /api/admin/products/:id`
- `DELETE /api/admin/products/:id`

There is no admin panel in the frontend.

## 6. Important before launch

1. Use a strong random `JWT_SECRET`.
2. Set `FRONTEND_ORIGIN` to the real HTTPS domain of the app.
3. Change the admin password immediately.
4. Put payment secrets only in `.env`/server secrets.
5. Configure real payment APIs + webhooks and verify signatures before marking an order as paid.
6. Put the API behind HTTPS and a domain such as `api.yourdomain.com`.
7. For a larger production store, move SQLite to PostgreSQL and uploads to object storage.

## 7. Frontend connection example

```js
const API_BASE_URL = 'https://api.yourdomain.com/api';

async function api(path, options={}) {
  const token = localStorage.getItem('zhuo_token');
  const headers = {'Content-Type':'application/json', ...(options.headers||{})};
  if(token) headers.Authorization = `Bearer ${token}`;
  const r = await fetch(API_BASE_URL + path, {...options, headers});
  const data = await r.json().catch(()=>({}));
  if(!r.ok) throw new Error(data.error || 'API error');
  return data;
}
```

Then login:

```js
const result = await api('/auth/login', {
  method:'POST',
  body:JSON.stringify({email,password})
});
localStorage.setItem('zhuo_token', result.token);
```


## Single-service production setup
The frontend is bundled in `public/index.html` and is served by the same Express server. The app uses `/api` on the current origin, so no frontend API URL needs to be edited after deployment. Admin UI is not bundled as a user-facing route; admin operations remain protected by backend JWT role checks.

## Important
The app does not fake successful payments. Checkout creates a `pending` order; a real payment provider must be configured and its verified webhook must mark the order/payment as paid before digital fulfillment is granted.

## Deploy as one app
1. Push this entire folder to GitHub.
2. On Render, create a Web Service from the repo.
3. Build: `npm install`.
4. Start: `npm start`.
5. Add `JWT_SECRET`, `ADMIN_EMAIL`, and `ADMIN_PASSWORD` as secret environment variables.
6. Deploy. Open the service URL; the frontend is served at `/` and the API is at `/api`.
7. Run `npm run seed` once in a shell after the first deploy if you need the initial catalog/admin account.

For a real production store, use persistent storage/PostgreSQL rather than ephemeral SQLite storage. Do not claim a payment is successful until a real provider webhook verifies it.
