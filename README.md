# Flutter Shoe Store

A cross-platform shoe shopping app built with Flutter, backed by a Node.js/Express REST API and MongoDB, with an in-app admin dashboard, VietQR bank-transfer payments and QR-based order tracking.

![Flutter](https://img.shields.io/badge/Flutter-02569B?logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-0175C2?logo=dart&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?logo=firebase&logoColor=black)

> Originally a mobile-development course project (HUTECH). The Flutter package is still named `store_app`.

## Features

**Customers**
- Browse products from the API and filter by category
- Product details, size selector, wishlist and shopping cart
- Place orders and view "My Orders"
- Pay by bank transfer with a generated VietQR code
- Order tracking screen with a Google Map and a shipping timeline
- Edit profile (name, phone, avatar via gallery upload or image URL) and shipping addresses
- Light/dark theme, English and Vietnamese (`en_US`, `vi_VN`) translations, in-app notifications
- Sign in with email/password (API), plus Firebase-based Google and Facebook sign-in

**Admin**
- Admin dashboard with four tabs: Products, Orders, Users, Tracking
- Create, edit and delete products, with image upload or image URL
- Change order status and change user roles
- Dashboard statistics endpoint (users, products, orders, revenue, orders by status)
- QR scanner screen for staff to log a parcel's location (tracking scan)

## Tech stack

| Layer | Technology |
| --- | --- |
| Mobile / web client | Flutter (Dart SDK `^3.9.2`), GetX (state, routing, DI), GetStorage, `http`, `file_picker`, `image_picker` |
| Maps, QR, UI | `google_maps_flutter`, `geolocator`, `mobile_scanner`, `qr_flutter`, `lottie`, `shimmer`, `cached_network_image`, `timeline_tile` |
| Firebase (client) | `firebase_core`, `firebase_auth`, `cloud_firestore`, `google_sign_in`, `flutter_facebook_auth` |
| API server | Node.js (ES modules), Express 4, Mongoose 8, JWT (`jsonwebtoken`), `bcryptjs`, Multer, `qrcode`, `node-fetch` |
| Database | MongoDB |
| Payments | VietQR QuickLink (vietqr.io) with a Casso-style webhook |

## Architecture

The app talks to two backends:

- **Node/Express + MongoDB** is the main API for products, users, orders, uploads, payments and tracking, authenticated with JWT.
- **Firebase** (Auth and Firestore) supports social sign-in and some per-user data such as wishlist and cart.

```
.
├── lib/                      # Flutter app
│   ├── controllers/          # GetX controllers (auth, cart, wishlist, theme, language, navigation)
│   ├── models/               # Product, user, tracking models
│   ├── repositories/         # API/Firestore access (products, orders, users, uploads, payments, tracking)
│   ├── services/             # ApiService (HTTP client), auth and Firestore services
│   ├── view/                 # Screens (home, cart, orders, tracking, QR, admin dashboard, ...)
│   ├── widgets/              # Reusable widgets
│   ├── features/ shipping_address/   # Feature-scoped orders, profile and address code
│   └── utils/                # Themes, translations, formatters, helpers
├── server/                   # Node/Express API
│   └── src/
│       ├── index.js          # App entry point, route mounting, /uploads static hosting
│       ├── routes/           # auth, products, orders, users, admin, uploads, payments_qr, tracking
│       ├── models/           # User, Product, Order, TrackingHistory (Mongoose)
│       ├── middleware/auth.js# JWT verification + admin-only guard
│       ├── utils/vietqr.js   # VietQR helpers
│       └── seed.js           # Demo data seeding
├── assets/                   # Images and Lottie animation
├── android/ ios/ web/ windows/ macos/ linux/   # Flutter platform folders
└── test/                     # Flutter tests
```

## Getting started

### Prerequisites

- [Flutter SDK](https://docs.flutter.dev/get-started/install) with Dart `^3.9.2`
- Node.js 20+ and npm
- A running MongoDB instance (local or hosted)
- Android Studio / an emulator or a physical device (for mobile)
- Optional: a Firebase project (for Google/Facebook sign-in and Firestore) and a Google Maps API key (for the tracking map)

### 1. Clone

```bash
git clone https://github.com/KimThanhTran/store_app.git
cd store_app
```

### 2. Run the API server

```bash
cd server
npm install
cp .env.example .env     # then edit the values (see below)
npm run seed             # optional: demo products, users and one order
npm run dev              # or: npm start
```

The server listens on `http://localhost:3000` by default. Check `http://localhost:3000/health`.

#### Environment variables (`server/.env`)

```env
PORT=3000
MONGO_URI=mongodb://127.0.0.1:27017/store_app
MONGO_DB=store_app
JWT_SECRET=replace_with_a_long_random_string

# VietQR payment (bank info used to build the QR code)
VIETQR_BANK_BIN=<your_bank_bin>
VIETQR_ACC_NUMBER=<your_account_number>
VIETQR_ACC_NAME=<ACCOUNT HOLDER NAME>
VIETQR_USE_VIETQR_IO=true
VIETQR_TEMPLATE=compact2

# Payment webhook (Casso/PayOS style)
QR_WEBHOOK_SECRET=<your_webhook_api_key>
MOCK_QR_WEBHOOK=false     # set true in development to allow POST /api/payments/qr/mockPaid
DEBUG=true
```

Only `MONGO_URI` (defaults to a local database) and `JWT_SECRET` are needed to run the core shop; the VietQR variables are needed only for QR payments.

#### Seeded demo accounts

`npm run seed` creates one admin and one regular user for local testing (see `server/src/seed.js` for the credentials). Change or remove them before deploying to any shared environment.

### 3. Run the Flutter app

```bash
# from the repository root
flutter pub get
flutter run
```

The API base URL is chosen automatically in `lib/services/api_service.dart`:

| Platform | Default base URL |
| --- | --- |
| Web, desktop, iOS simulator | `http://localhost:3000` |
| Android emulator | `http://10.0.2.2:3000` |

To target another host (for example a physical device on your LAN):

```bash
flutter run --dart-define=API_BASE_URL=http://192.168.1.10:3000
```

### 4. Firebase, Facebook and Google Maps (optional)

- Firebase options live in `lib/firebase_options.dart`, and the Android config in `android/app/google-services.json`. To use your own project, regenerate them with the FlutterFire CLI (`flutterfire configure`).
- Facebook identifiers are read from `lib/utils/app_secrets.dart`.
- The Android Google Maps key is declared in `android/app/src/main/AndroidManifest.xml`.

The Facebook client token (`strings.xml`, `app_secrets.dart`) and the Google Maps key (`AndroidManifest.xml`) are committed as placeholders (`YOUR_FACEBOOK_CLIENT_TOKEN`, `YOUR_GOOGLE_MAPS_API_KEY`); put your own values in locally and do not commit them. Replace all of these with your own values before building; email/password login against the API does not depend on them.

## API overview

Base URL: `http://localhost:3000`. Protected routes expect `Authorization: Bearer <JWT>`.

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| GET | `/health` | Public | Server and MongoDB connection state |
| POST | `/api/auth/register` | Public | Register with email and password |
| POST | `/api/auth/login` | Public | Login, returns JWT (7-day expiry) and user |
| POST | `/api/auth/social-token` | Public | Exchange a Firebase UID/email for an API token |
| GET | `/api/products` | Public | List products (`?category=`, `?limit=`) |
| POST | `/api/products` | Admin | Create product |
| PUT | `/api/products/:id` | Admin | Update product |
| DELETE | `/api/products/:id` | Admin | Delete product |
| POST | `/api/orders` | User | Create an order |
| GET | `/api/orders/my` | User | Current user's orders |
| GET | `/api/orders` | Admin | All orders |
| PATCH | `/api/orders/:id/status` | Admin | Set status: `pending`, `processing`, `shipped`, `delivered`, `cancelled` |
| GET | `/api/users` | Admin | List users |
| GET / PUT | `/api/users/me` | User | Get / update own profile (name, phone, avatar URL) |
| PUT | `/api/users/:id` | Admin | Update a user (including role) |
| GET | `/api/admin/stats` | Admin | Totals, revenue and orders by status |
| POST | `/api/uploads/images` | Admin | Upload an image (multipart field `image`) |
| POST | `/api/payments/qr/create` | Public | Create a VietQR code for `orderId` and `amountVnd` |
| GET | `/api/payments/qr/status?orderId=` | Public | Payment intent status (`pending`, `paid`, `expired`) |
| GET / POST | `/api/payments/qr/webhook` | Public | Bank-transfer webhook (GET is a verification ping) |
| POST | `/api/payments/qr/mockPaid` | Dev only | Mark an intent paid when `MOCK_QR_WEBHOOK=true` |
| POST | `/api/tracking/scan` | Staff | Log a parcel scan (location, status, notes) |
| GET | `/api/tracking/:trackingId` | User / Staff | Tracking history for an order |
| GET | `/api/tracking/order/:orderId` | User / Staff | Tracking history by order id |

Uploaded files are served from `/uploads/...`.

## Roles and authentication

- JWT payload contains the user id and a numeric `role`: `0` = admin, `1` = customer.
- Admin routes use the `authRequired` + `adminOnly` middleware.
- Customers can only read tracking data for their own orders; scanning is limited to non-customer roles (staff).
- After login the app routes admin users to the admin dashboard.
- Passwords are hashed with bcrypt. Social sign-in users are created without a password and identified by their Firebase UID.

## Payments and tracking notes

- QR payment intents are stored in server memory with a 15-minute expiry (the source marks this as demo-only), so they reset when the server restarts.
- Webhook matching is done on transfer amount and an `ORDER-<orderId>` transfer description. Signature verification is present in code but currently commented out.
- Each order receives a generated `trackingId` (`TRK...`). Scans append to a tracking history and can advance the order status.

## Testing

```bash
flutter test
```

## License

No license file is included in this repository.
