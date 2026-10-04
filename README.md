# Sulfa Media Store — Nokos Digital

Stack:
- React + Vite
- Firebase Authentication
- Firestore
- Firebase Cloud Functions v2
- Cloudflare Pages or Firebase Hosting for the frontend

## 1. Install

```bash
npm install
cd functions
npm install
cd ..
```

## 2. Firebase

Enable:
- Authentication > Email/Password
- Firestore Database
- Cloud Functions

Deploy rules/functions:

```bash
firebase login
firebase use web-nokosgur
firebase deploy --only firestore:rules
cd functions
npm run build
cd ..
firebase deploy --only functions
```

## 3. Run frontend

```bash
npm run dev
```

## 4. Cloudflare Pages

Build command:

```bash
npm run build
```

Output directory:

```text
dist
```

Add SPA fallback/rewrite so routes resolve to `/index.html`.

## 5. First developer account

After registering your own account, set its role once from a trusted Admin SDK script or Firebase console:

```text
users/{UID}
role: developer
balance: 0
purchaseCount: 0
```

Do not expose role-editing UI to normal users.

## 6. Product model

```text
products/{productId}
  name
  description
  price
  stockCount
  active

products/{productId}/inventory/{inventoryId}
  value
  status: ready|sold
  soldTo
  soldAt
```

Inventory values are only returned by the server-side purchase function.

## 7. Deposit model

```text
deposits/{depositId}
  userId
  amount
  status: pending|paid|expired|proof_submitted
  providerReference
  createdAt
  paidAt
```

The included frontend displays your uploaded QRIS after a user enters a deposit amount.

The uploaded QRIS is a static merchant QR. A static QR cannot by itself prove that a specific deposit was paid, match a unique order, or trigger an automatic balance credit. Automatic balance credit requires a QRIS/payment provider that creates a dynamic payment and sends a signed webhook to `qrisWebhook`.

## 8. Automatic deposit flow

```text
Customer enters amount
        |
        v
createDeposit()
        |
        v
Payment provider creates dynamic QRIS
        |
        v
QR returned to frontend
        |
        v
Customer pays
        |
        v
Provider webhook -> qrisWebhook
        |
        v
Firestore transaction
  deposit = paid
  user.balance += amount
        |
        v
Customer sees new balance
```

Replace `qrImagePath` in `createDeposit` with the dynamic QR image/QR string returned by your provider.

## 9. Role hierarchy

```text
pembeli
  └── after 5 successful purchases -> pelanggan

admin
  ├── product management
  ├── stock management
  └── store information

developer
  ├── everything admin can do
  └── deployment / maintenance access
```

The hierarchy is enforced server-side in Cloud Functions. The client only controls what is displayed.
