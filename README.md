# Mtaani Market: complete online store

Front end, backend, phone-OTP login, orders, payments, admin dashboard and legal pages for selling clothes, shoes and crafts in Kenya, Uganda, Tanzania, Rwanda, Ethiopia and South Sudan.

No npm packages. You only need **Node.js 22.13 or newer** (https://nodejs.org).

## Try it on your computer (2 minutes)
```
cd mtaani-market
cp .env.example .env        # Windows: copy .env.example .env
node server.js
```
- Shop: http://localhost:3000   Admin: http://localhost:3000/admin
- Dev admin password is printed in the terminal (`admin12345` unless you set ADMIN_PASSWORD).
- In development, login codes show on screen (and in the terminal) and payments use a fake test page, so nothing real is sent or charged.
- Run the 33 automatic checks: `npm test`

## What is where
| File | What it does |
|---|---|
| `server.js` | Backend: login codes, sessions, orders, stock, payments, admin API, SQLite database in `data/store.db` |
| `public/index.html`, `app.js`, `styles.css` | The shop |
| `public/admin.html`, `admin.js` | Admin dashboard: orders (change status, customer gets an SMS when shipped), products (add, edit, remove, stock, photos, prices) |
| `public/privacy.html`, `terms.html`, `returns.html` | Legal page templates, **have a local lawyer review them** |
| `.env.example` | All settings |
| `test.js` | Automatic checks |

## Going live: step by step

### 1. Accounts you need (all free to open)
1. **Africa's Talking** (africastalking.com): sends SMS in all six countries. Make an app, top up airtime credit, copy the username and API key. Ask for a Sender ID if you want your shop name on messages (approval can take days). Test first with username `sandbox`.
2. **Flutterwave** (flutterwave.com): takes M-Pesa, MTN MoMo, Airtel Money and cards in Kenya, Uganda, Tanzania and Rwanda. Complete business verification (KYC), then copy the **secret key** from Settings > API.
3. A **host** that runs Node with a persistent disk, for example Render, Railway, Fly.io or a small VPS. Use HTTPS (hosts do this for you) and your own domain.

### 2. Settings (environment variables on your host, same names as `.env.example`)
`NODE_ENV=production`, `BASE_URL=https://your-domain`, `SESSION_SECRET` (32+ random characters), `ADMIN_PASSWORD` (10+ characters), `TRUST_PROXY=1`, `DATA_DIR` (a path on the persistent disk), `AT_USERNAME`, `AT_API_KEY`, `AT_SENDER_ID`, `FLW_SECRET_KEY`, `FLW_SECRET_HASH`, `BUSINESS_NAME`, `BUSINESS_EMAIL`, `BUSINESS_ADDRESS`.
In production the server refuses to start if a required secret is missing, and DEV_SHOW_OTP is ignored.

### 3. Flutterwave webhook
Dashboard > Settings > Webhooks: URL `https://your-domain/api/webhooks/flutterwave`, and put the same text as `FLW_SECRET_HASH` in "Secret hash". The server never trusts the webhook alone: it re-checks every payment with Flutterwave (amount, currency, reference) before marking an order paid.

### 4. Set real prices and rates
- Prices are stored in **Kenyan shillings**. Open `server.js` and edit the `COUNTRIES` list: exchange rate `r`, rounding `st`, delivery fee `fee` and free-delivery limit `free` per country. Check rates regularly; South Sudan's pound especially.
- Add your real products, photos, sizes and stock in `/admin`. Photos are https links (host them on Cloudinary, Imgur or your own site).
- Confirm the mobile number prefixes in `COUNTRIES` (`re`) with your SMS provider, especially for South Sudan.

### 5. Ethiopia and South Sudan
Flutterwave does not charge Ethiopian birr or South Sudanese pounds, so those countries use **pay on delivery** only. To add Telebirr, MTN MoMo South Sudan or similar later, you need that provider's merchant API; the place to add it is the `/api/orders/:id/pay` route in `server.js`.

### 6. Business basics before the first sale
- Register the business and tax numbers where you operate; follow your country's e-commerce and consumer rules.
- Register as a data controller/processor if your country requires it (Kenya: Office of the Data Protection Commissioner).
- Edit the three legal pages and delete the yellow template notice.
- Line up delivery riders or a courier, and decide your real delivery areas and times.
- Back up `data/store.db` regularly (copy the file; your host may offer disk snapshots).
- Place a real test order end to end (small amount) before announcing.

## How it works (short)
- **Login:** name + phone, number checked per country, 6-digit code stored only as a keyed hash, valid 5 minutes, 5 tries, rate limited by phone and IP, then a 30-day HttpOnly session cookie.
- **Orders:** the browser only sends product ids, sizes and quantities. The server looks up prices, converts currency, checks stock and computes the total. Stock is reserved at order time; cancelled or unpaid (2 hours) orders restock.
- **Payments:** Flutterwave hosted checkout; confirmed server-side. Pay on delivery orders are confirmed immediately.
- **Admin:** password login (rate limited), signed 12-hour cookie.

## Limits of this version (honest list)
- Built and tested here with automatic checks, but the **Africa's Talking and Flutterwave calls were not run against the live services** (they need your accounts). Test in their sandboxes before real money.
- SQLite on one server suits a small to medium shop. If you outgrow it, move to Postgres and a managed host.
- No customer email notifications, coupons, refunds API, or multi-staff admin accounts yet.
