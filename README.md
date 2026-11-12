# Binhi · Non-Profit Agricultural Initiative & Payment Integration

Welcome to **Binhi**, a non-profit agricultural initiative dedicated to empowering smallholder farmers in the Philippines through certified high-yield seed distribution, guaranteed fair-trade market links, and community resilience workshops.

This repository features a complete, secure, end-to-end online donation portal integrated with:
- **PayMongo**: Philippine e-wallets (GCash, Maya, GrabPay), local bank cards, and Billease.
- **Stripe (`@stripe/react-stripe-js` & `@stripe/stripe-js`)**: Embedded Stripe Elements credit/debit card processing (Visa, Mastercard, American Express, Apple Pay, Google Pay).

---

## 🌾 Tech Stack & Architecture

- **Frontend**: React 19, TypeScript, Vite, custom responsive CSS design system (`binhi.css`, `donation.css`).
- **Payment Gateways**:
  - **PayMongo**: v1 Hosted Checkout Sessions + Sandbox Simulator.
  - **Stripe**: `@stripe/react-stripe-js` embedded card elements & PaymentIntents.
- **Backend API**: Node.js v24 (using native TypeScript strip-types), Express 5, CORS, and Express Rate Limit.
- **Database**: SQLite using Node 24 native `node:sqlite` (`DatabaseSync`), persistent WAL mode in `./data/donations.sqlite`.
- **Testing**: Node.js native test runner (`node:test`, `node:assert`).
- **Security**: Server-side secret key isolation, rate limiting on checkout endpoints, raw body HMAC SHA256 webhook signature verification with replay attack prevention, and security HTTP headers (`X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, `Referrer-Policy: strict-origin-when-cross-origin`).

---

## 🚀 Quick Start (Local Setup)

### 1. Install Dependencies
```bash
npm install
```

### 2. Configure Environment Variables
Copy the example environment configuration to `.env`:
```bash
cp .env.example .env
```

Review `.env`:
```env
PORT=3001
NODE_ENV=development
FRONTEND_URL=http://localhost:5173

# PayMongo Test Credentials
PAYMONGO_SECRET_KEY=sk_test_replace_with_your_test_secret_key
PAYMONGO_PUBLIC_KEY=pk_test_replace_with_your_test_public_key
PAYMONGO_WEBHOOK_SECRET=whsk_test_demo_signing_secret

DATABASE_PATH=./data/donations.sqlite
```
> **Note on Sandbox Simulator Mode**: When `PAYMONGO_SECRET_KEY` contains the placeholder string, the system automatically activates the **PayMongo Test Checkout Simulator**, allowing you to test GCash, Maya, Card, Failed, and Cancelled donations locally without an active PayMongo merchant account. Once you provide real `sk_test_...` keys, it immediately makes live calls to PayMongo's official API!

### 3. Run Automated Tests
```bash
npm test
```
Runs all 22 unit and integration tests covering:
- Client and server-side amount and email validation.
- Checkout session creation.
- PayMongo HMAC SHA256 webhook signature verification (including tampered payload rejection and replay drift protection).
- Webhook idempotency and database state transitions.

### 4. Run the Full Application Locally
You can run the frontend and backend in parallel:

**Terminal 1 (Backend API):**
```bash
npm run server
```
Server starts on `http://localhost:3001`.

**Terminal 2 (Frontend Dev Server):**
```bash
npm run dev
```
Vite starts on `http://localhost:5173` (requests to `/api` are automatically proxied to port 3001).

**Or Single-Command Production Run:**
```bash
npm start
```
Builds the client and serves both API and static assets from `http://localhost:3001`.

---

## 💳 PayMongo Payment & Webhook Setup

### Obtaining Test API Keys
1. Register an account at [PayMongo Dashboard](https://dashboard.paymongo.com/).
2. Toggle the switch in the top navigation to **Test Mode**.
3. Navigate to **Developers** → **API Keys**.
4. Copy your **Public Key** (`pk_test_...`) and **Secret Key** (`sk_test_...`).
5. Paste them into your `.env` file.

### Registering Webhooks for Local Testing
To receive real webhook callbacks from PayMongo when running locally:
1. Expose your local port 3001 using a tunnel (e.g. ngrok or Localtunnel):
   ```bash
   ngrok http 3001
   ```
2. In PayMongo Dashboard, go to **Developers** → **Webhooks** → **Add Webhook**.
3. Set the Webhook URL to:
   ```
   https://<your-ngrok-subdomain>.ngrok-free.app/api/webhooks/paymongo
   ```
4. Select the events to listen for:
   - `checkout_session.payment.paid`
   - `payment.paid`
   - `payment.failed`
5. Copy the generated **Webhook Signing Secret** (`whsk_...`) and set it in `.env` as `PAYMONGO_WEBHOOK_SECRET`.

---

## 📋 Needs Your Input (Remaining `[REPLACE: ...]` Placeholders)

The following items are marked with `[REPLACE: ...]` in the source code for the organization owner to provide:

1. `[REPLACE: org legal name]` — Registered corporate name with SEC (e.g., *Binhi Agricultural Development Foundation, Inc.*).
2. `[REPLACE: SEC registration number]` — Securities and Exchange Commission non-stock non-profit registration number.
3. `[REPLACE: BIR tax identification number (TIN)]` — Bureau of Internal Revenue Tax Identification Number.
4. `[REPLACE: official contact email]` — Official donation and support email (e.g., `donations@binhi.ph`).
5. `[REPLACE: contact phone]` — Official contact number (e.g., `+63 (2) 8XXX-XXXX` or mobile).
6. `[REPLACE: registered office address]` — Official business address in the Philippines.
7. `[REPLACE: Facebook URL]` — Official Facebook page URL.
8. `[REPLACE: Instagram URL]` — Official Instagram account URL.
9. `[REPLACE: LinkedIn URL]` — Official LinkedIn organization URL.

---

## 🚀 Go-Live Checklist (Philippine Non-Profit Donations)

Before accepting real money from the public:

- [ ] **PayMongo Business Verification**: Complete PayMongo merchant onboarding by submitting SEC Certificate of Incorporation, Articles of Incorporation, BIR Certificate of Registration (Form 2303), and valid IDs of authorized signatories.
- [ ] **Switch API Keys**: Update `.env` with Live credentials (`sk_live_...` and `pk_live_...`).
- [ ] **Live Webhook Endpoint**: Register production webhook URL `https://yourdomain.ph/api/webhooks/paymongo` in the PayMongo live dashboard.
- [ ] **HTTPS Enforced**: Ensure all traffic is served over TLS/SSL (mandatory for payment processing).
- [ ] **DSWD Authority / Permit**: Secure public solicitation authority under Republic Act No. 4373 / DSWD guidelines if conducting active nationwide solicitations.
- [ ] **BIR Certificate of Donation & PCNC Certification**: If donors require tax deductibility under the Tax Code (Section 34(H)), register with the Philippine Council for NGO Certification (PCNC) and obtain BIR Certificate of Tax Exemption.
- [ ] **Live Mailer Configuration**: Plug your SMTP / SendGrid / Resend credentials into `server/email.ts` to automatically dispatch electronic tax receipts to donors.
