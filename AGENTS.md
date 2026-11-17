# Project Instructions & Rules for AI Agents

## 1. Tech Stack & Architecture
- **Framework:** Next.js (App Router)
- **Language:** TypeScript (Strict mode enabled)
- **Styling:** Tailwind CSS (v4)
- **Directory Structure:**
  - `src/app/`: Routes, layouts, and pages (`page.tsx`, `layout.tsx`).
  - `src/components/`: Reusable UI components.
  - `src/lib/` or `src/utils/`: Helper functions and API clients.

## 2. Environment Variables & API Versioning
- **Base Domain:** `https://ecommerce.routemisr.com`
- **API v1 Base URL:** `process.env.NEXT_PUBLIC_API_URL` (`https://ecommerce.routemisr.com/api/v1`)
- **API v2 Base URL:** `process.env.NEXT_PUBLIC_API_V2_URL` (`https://ecommerce.routemisr.com/api/v2`)
- **Important Versioning Rule:**
  - **Use `/api/v2` ONLY for:** All **Cart** endpoints and **Create Cash Order** (`POST /api/v2/orders/{cartId}`).
  - **Use `/api/v1` for:** **Online Payment Checkout Session** (`POST /api/v1/orders/checkout-session/{cartId}`), **Fetching Orders** (`GET`), Auth, Products, Categories, SubCategories, Brands, Wishlist, and Addresses.

## 3. FreshCart Complete API Endpoints Reference
Every request to protected routes (Cart, Wishlist, Orders, Addresses, User Profile) requires a token in headers: `token: localStorage.getItem('userToken')` or sent via cookies.

---

### A. Auth & User Profile — `[API v1]` (المصادقة وبيانات الحساب)
- **Signup:** `POST /api/v1/auth/signup`
  - Body: `name`, `email`, `password`, `rePassword`, `phone`
- **Signin (Login):** `POST /api/v1/auth/signin`
  - Body: `email`, `password`
  - Response returns: `token` and user object.
- **Forgot Password:** `POST /api/v1/auth/forgotPasswords`
  - Body: `email`
- **Verify Reset Code:** `POST /api/v1/auth/verifyResetCode`
  - Body: `resetCode`
- **Reset Password:** `PUT /api/v1/auth/resetPassword`
  - Body: `email`, `newPassword`
- **Verify Token:** `GET /api/v1/auth/verifyToken`
  - Headers: `token`
  - Response returns: `decoded.id` (the exact `userId` needed for fetching user orders).
- **Update Logged User Password:** `PUT /api/v1/users/changeMyPassword`
  - Headers: `token`
  - Body: `{ "currentPassword": "...", "password": "...", "rePassword": "..." }`
- **Update Logged User Data:** `PUT /api/v1/users/updateMe/`
  - Headers: `token`
  - Body: `{ "name": "...", "email": "...", "phone": "..." }`

### B. Products, Categories, SubCategories & Brands — `[API v1]` (المنتجات والأقسام والماركات)
- **Get All Products:** `GET /api/v1/products`
  - Query params (optional): `page`, `limit`, `category[in]`, `brand`, `sort`, `keyword`, `price[gte]`, `price[lte]`
- **Get Specific Product:** `GET /api/v1/products/{id}`
- **Get All Categories:** `GET /api/v1/categories`
- **Get Specific Category:** `GET /api/v1/categories/{id}`
- **Get All SubCategories:** `GET /api/v1/subcategories`
- **Get Specific SubCategory:** `GET /api/v1/subcategories/{id}`
- **Get SubCategories on Category:** `GET /api/v1/categories/{categoryId}/subcategories`
- **Get All Brands:** `GET /api/v1/brands`
- **Get Specific Brand:** `GET /api/v1/brands/{id}`

### C. Wishlist — `[API v1]` (المفضلة - محمية بالتوكن)
- **Get Logged User Wishlist:** `GET /api/v1/wishlist`
  - Headers: `token`
- **Add to Wishlist:** `POST /api/v1/wishlist`
  - Headers: `token`
  - Body: `{ "productId": "..." }`
- **Remove from Wishlist:** `DELETE /api/v1/wishlist/{productId}`
  - Headers: `token`

### D. User Addresses — `[API v1]` (عناوين المستخدم - محمية بالتوكن)
- **Get Logged User Addresses:** `GET /api/v1/addresses`
  - Headers: `token`
- **Add Address:** `POST /api/v1/addresses`
  - Headers: `token`
  - Body: `{ "name": "Home", "details": "...", "phone": "...", "city": "..." }`
- **Get Specific Address:** `GET /api/v1/addresses/{addressId}`
  - Headers: `token`
- **Remove Address:** `DELETE /api/v1/addresses/{addressId}`
  - Headers: `token`

---

### E. Cart — `[API v2]` (سلة المشتريات - الإصدار الثاني)
- **Get Logged User Cart:** `GET /api/v2/cart`
  - Headers: `token`
- **Add Product to Cart:** `POST /api/v2/cart`
  - Headers: `token`
  - Body: `{ "productId": "..." }`
- **Update Cart Product Quantity:** `PUT /api/v2/cart/{productId}`
  - Headers: `token`
  - Body: `{ "count": 2 }`
- **Remove Specific Cart Item:** `DELETE /api/v2/cart/{productId}`
  - Headers: `token`
- **Clear User Cart:** `DELETE /api/v2/cart`
  - Headers: `token`
- **Apply Coupon to Cart:** `PUT /api/v2/cart/applyCoupon`
  - Headers: `token`
  - Body: `{ "couponName": "..." }`

---

### F. Checkout & Orders (الدفع والطلبات - انتبه لاختلاف الإصدار)
- **Create Cash Order — `[API v2]`:** `POST /api/v2/orders/{cartId}`
  - Headers: `token`
  - Body: `{ "shippingAddress": { "details": "...", "phone": "...", "city": "..." } }`
- **Get Checkout Session (Online Payment) — `[API v1]`:** `POST /api/v1/orders/checkout-session/{cartId}?url=http://localhost:3000`
  - Headers: `token`
  - Body: `{ "shippingAddress": { "details": "...", "phone": "...", "city": "..." } }`
  - Note: Pass `window.location.origin` as the `url` query parameter (e.g., `?url=http://localhost:3000`) so Stripe redirects back to `/allorders` on the same host.
- **Get User Orders — `[API v1]`:** `GET /api/v1/orders/user/{userId}`
  - Note: `userId` MUST be the user's ID decoded from the JWT `userToken` (`payload.id`), NOT `cartId`.
- **Get All Orders — `[API v1]`:** `GET /api/v1/orders/`


### G. Reviews — `[API v1]` (التقييمات والتعليقات - بعضها محمي بالتوكن)
- **Get Product Reviews:** `GET /api/v1/products/{productId}/reviews`
- **Create Product Review:** `POST /api/v1/products/{productId}/reviews`
  - Headers: `token`
  - Body: `{ "title": "...", "ratings": 5 }`
- **Get All Reviews:** `GET /api/v1/reviews`
- **Get Specific Review:** `GET /api/v1/reviews/{reviewId}`
- **Update Review:** `PUT /api/v1/reviews/{reviewId}`
  - Headers: `token`
  - Body: `{ "title": "...", "ratings": 5 }`
- **Delete Review:** `DELETE /api/v1/reviews/{reviewId}`
  - Headers: `token`



## 4. Coding & Architecture Rules
- Use Client Components (`'use client'`) with `useEffect` when fetching user-specific protected data that depends on `localStorage` tokens (Cart, Wishlist, User Orders) to avoid SSR hydration mismatches and slow server timeouts.
- Always use Tailwind CSS for styling to mirror the FreshCart UI.
- Never use `any` in TypeScript; define strict interfaces for Products, Categories, Cart, Orders, and User responses.
- When building API helper functions, explicitly route:
  - All Cart calls + `POST` Cash Order (`POST /orders/{cartId}`) to `v2` (`https://ecommerce.routemisr.com/api/v2`).
  - `POST` Online Checkout Session (`POST /orders/checkout-session/{cartId}`), `GET` User Orders (`GET /orders/user/{userId}`), and all other endpoints to `v1` (`https://ecommerce.routemisr.com/api/v1`).