# Razorpay Payment Integration Setup

This document explains how to set up Razorpay payment integration for the Craft pricing page.

## Prerequisites

1. A Razorpay account (sign up at https://razorpay.com)
2. Razorpay API keys (Key ID and Key Secret)

## Setup Instructions

### 1. Install Razorpay Package

```bash
npm install razorpay
```

### 2. Configure Environment Variables

Add the following environment variables to your `.env.local` file:

```env
# Razorpay Configuration
RAZORPAY_KEY_ID=your_razorpay_key_id
RAZORPAY_KEY_SECRET=your_razorpay_key_secret
NEXT_PUBLIC_RAZORPAY_KEY_ID=your_razorpay_key_id
```

**Important:**

- Get these keys from your Razorpay Dashboard → Settings → API Keys
- Use **Test Mode** keys for development
- Use **Live Mode** keys for production
- Never commit your `.env.local` file to version control

### 3. Test the Payment Flow

1. Start your development server:

   ```bash
   npm run dev
   ```

2. Navigate to `/pricing` page

3. Click on "Start Premium" button

4. Complete the payment using Razorpay test cards:
   - **Test Card:** 4111 1111 1111 1111
   - **CVV:** Any 3 digits
   - **Expiry:** Any future date

### 4. Verify Payment

After successful payment:

- The payment is verified using Razorpay signature verification
- User subscription status should be updated in the database
- User is redirected to the dashboard

## Features

### Currency Detection

- Automatically detects user location using IP geolocation
- Shows prices in INR for Indian users
- Shows prices in USD for all other users

### Payment Security

- All payments are processed securely through Razorpay
- Payment signatures are verified on the backend
- No sensitive payment information is stored

### Supported Payment Methods

- Credit Cards (Visa, MasterCard, American Express, etc.)
- Debit Cards
- UPI (for Indian users)
- Net Banking (for Indian users)
- Wallets (Paytm, PhonePe, etc.)

## API Endpoints

### Create Order

**POST** `/api/payment/create-order`

Creates a new Razorpay order for payment processing.

**Request Body:**

```json
{
  "amount": 50000,
  "currency": "INR",
  "plan": "Premium"
}
```

**Response:**

```json
{
  "orderId": "order_xxxxxxxxxxxxx",
  "amount": 50000,
  "currency": "INR"
}
```

### Verify Payment

**POST** `/api/payment/verify`

Verifies the payment signature after successful payment.

**Request Body:**

```json
{
  "orderId": "order_xxxxxxxxxxxxx",
  "paymentId": "pay_xxxxxxxxxxxxx",
  "signature": "xxxxxxxxxxxxx"
}
```

**Response:**

```json
{
  "verified": true,
  "message": "Payment verified successfully"
}
```

## Database Updates Required

After successful payment verification, you should:

1. Update user's subscription plan in the database
2. Set subscription start and end dates
3. Grant access to premium features
4. Send confirmation email to the user

Example Prisma schema update:

```prisma
model User {
  id                String    @id @default(cuid())
  email             String    @unique
  subscriptionPlan  String    @default("Free") // "Free", "Premium", "Enterprise"
  subscriptionStart DateTime?
  subscriptionEnd   DateTime?
  razorpayCustomerId String?
  razorpayPaymentId  String?
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
}
```

## Production Checklist

Before going live:

- [ ] Switch to Razorpay Live Mode keys
- [ ] Implement proper error handling
- [ ] Add payment logging for audit trails
- [ ] Set up webhook handlers for subscription events
- [ ] Implement refund handling
- [ ] Add email notifications for successful payments
- [ ] Test with real payment methods
- [ ] Implement subscription renewal logic
- [ ] Add invoice generation
- [ ] Set up monitoring and alerts

## Troubleshooting

### Payment Modal Not Opening

- Check if Razorpay script is loaded
- Verify `NEXT_PUBLIC_RAZORPAY_KEY_ID` is set correctly
- Check browser console for errors

### Payment Verification Failed

- Ensure `RAZORPAY_KEY_SECRET` is correct
- Check if order ID and payment ID are valid
- Verify signature generation logic

### Currency Not Detected

- IP geolocation service may be blocked
- Default to USD if location detection fails
- Consider adding manual currency selector

## Support

For Razorpay integration issues:

- Razorpay Documentation: https://razorpay.com/docs/
- Razorpay Support: https://razorpay.com/support/

For Craft-specific issues:

- Create an issue on GitHub
- Contact support@craft.tech
