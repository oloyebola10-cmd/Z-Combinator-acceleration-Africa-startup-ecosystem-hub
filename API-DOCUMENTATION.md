# API Documentation - Z-Combinator Africa Ecosystem

## Overview

The Z-Combinator Africa Ecosystem provides comprehensive API access to all platform services, enabling developers and partners to integrate with shop.africa, oloyepay, oloyebot, film village, fund Holding, and other ecosystem components.

## Base URL

```
Production: https://api.zcombinator.africa/v1
Sandbox: https://sandbox-api.zcombinator.africa/v1
```

## Authentication

### API Keys
All API requests require authentication using API keys passed in the request header:

```http
Authorization: Bearer YOUR_API_KEY
```

### OAuth 2.0
For user-specific operations, OAuth 2.0 authentication is required:

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
```

### Getting API Keys
1. Sign up at https://developer.zcombinator.africa
2. Create a new application
3. Generate API keys (sandbox and production)
4. Store keys securely (never commit to source control)

## Rate Limiting

- **Free Tier**: 1,000 requests/hour
- **Starter Tier**: 10,000 requests/hour
- **Business Tier**: 100,000 requests/hour
- **Enterprise Tier**: Custom limits

Rate limit headers:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1609459200
```

## Common Response Format

### Success Response
```json
{
  "success": true,
  "data": { },
  "message": "Operation completed successfully",
  "timestamp": "2025-12-31T05:00:00Z"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Invalid request parameters",
    "details": {}
  },
  "timestamp": "2025-12-31T05:00:00Z"
}
```

## API Endpoints

### 1. oloyepay Payment Processing API

#### Initialize Payment
Create a new payment transaction.

```http
POST /payments/initialize
```

**Request Body:**
```json
{
  "amount": 1000.00,
  "currency": "KES",
  "payment_method": "mobile_money",
  "mobile_money": {
    "provider": "mpesa",
    "phone_number": "+254712345678"
  },
  "customer": {
    "email": "customer@example.com",
    "name": "John Doe"
  },
  "callback_url": "https://yoursite.com/payment/callback",
  "reference": "ORDER-123456",
  "description": "Payment for order #123456"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "transaction_id": "txn_abc123xyz",
    "status": "pending",
    "payment_url": "https://pay.oloyepay.africa/txn_abc123xyz",
    "expires_at": "2025-12-31T05:30:00Z"
  }
}
```

#### Verify Payment
Check the status of a payment transaction.

```http
GET /payments/{transaction_id}/verify
```

**Response:**
```json
{
  "success": true,
  "data": {
    "transaction_id": "txn_abc123xyz",
    "status": "success",
    "amount": 1000.00,
    "currency": "KES",
    "payment_method": "mobile_money",
    "paid_at": "2025-12-31T05:05:00Z",
    "reference": "ORDER-123456"
  }
}
```

#### List Transactions
Retrieve a list of transactions.

```http
GET /payments/transactions?page=1&limit=50
```

**Query Parameters:**
- `page` (optional): Page number, default 1
- `limit` (optional): Results per page, default 50, max 100
- `status` (optional): Filter by status (pending, success, failed)
- `from_date` (optional): Start date (ISO 8601)
- `to_date` (optional): End date (ISO 8601)

#### Refund Payment
Process a refund for a transaction.

```http
POST /payments/{transaction_id}/refund
```

**Request Body:**
```json
{
  "amount": 500.00,
  "reason": "Customer requested partial refund"
}
```

### 2. shop.africa E-Commerce API

#### List Products
Retrieve products from the marketplace.

```http
GET /shop/products?category=electronics&page=1&limit=20
```

**Query Parameters:**
- `category` (optional): Filter by category
- `search` (optional): Search query
- `min_price` (optional): Minimum price
- `max_price` (optional): Maximum price
- `page` (optional): Page number
- `limit` (optional): Results per page

**Response:**
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "prod_123",
        "name": "Smartphone XYZ",
        "description": "High-quality smartphone",
        "price": 25000.00,
        "currency": "KES",
        "category": "electronics",
        "images": ["https://cdn.shop.africa/prod_123_1.jpg"],
        "in_stock": true,
        "vendor": {
          "id": "vendor_456",
          "name": "Tech Store Kenya"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8
    }
  }
}
```

#### Create Order
Place a new order.

```http
POST /shop/orders
```

**Request Body:**
```json
{
  "items": [
    {
      "product_id": "prod_123",
      "quantity": 1,
      "price": 25000.00
    }
  ],
  "customer": {
    "name": "Jane Doe",
    "email": "jane@example.com",
    "phone": "+254712345678"
  },
  "shipping_address": {
    "street": "123 Main Street",
    "city": "Nairobi",
    "country": "Kenya",
    "postal_code": "00100"
  },
  "payment_method": "mobile_money"
}
```

#### Track Order
Get order status and tracking information.

```http
GET /shop/orders/{order_id}
```

### 3. oloyebot AI Assistant API

#### Send Message
Interact with the AI assistant.

```http
POST /bot/chat
```

**Request Body:**
```json
{
  "message": "What is the status of my order?",
  "session_id": "session_abc123",
  "context": {
    "user_id": "user_456",
    "platform": "shop.africa"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "reply": "I can help you check your order status. Please provide your order number.",
    "session_id": "session_abc123",
    "intent": "order_tracking",
    "confidence": 0.95
  }
}
```

#### Get Bot Analytics
Retrieve conversation analytics.

```http
GET /bot/analytics?from_date=2025-01-01&to_date=2025-12-31
```

### 4. film village Content API

#### List Content
Browse available content.

```http
GET /content/list?type=movie&genre=drama
```

**Query Parameters:**
- `type` (optional): Content type (movie, series, documentary)
- `genre` (optional): Genre filter
- `language` (optional): Language filter
- `page` (optional): Page number
- `limit` (optional): Results per page

#### Stream Content
Get streaming URL for purchased content.

```http
POST /content/{content_id}/stream
```

**Response:**
```json
{
  "success": true,
  "data": {
    "stream_url": "https://stream.filmvillage.africa/content_123.m3u8",
    "expires_at": "2025-12-31T23:59:59Z",
    "quality_options": ["360p", "720p", "1080p"]
  }
}
```

### 5. fund Holding Investment API

#### List Investment Opportunities
Browse available investment opportunities.

```http
GET /investments/opportunities
```

#### Create Investment
Make a new investment.

```http
POST /investments/create
```

**Request Body:**
```json
{
  "opportunity_id": "opp_123",
  "amount": 50000.00,
  "currency": "KES",
  "investor": {
    "name": "John Investor",
    "email": "investor@example.com"
  }
}
```

#### Get Portfolio
Retrieve investor portfolio.

```http
GET /investments/portfolio/{investor_id}
```

### 6. oloyeDB Ledger API

#### Query Ledger
Search transaction records.

```http
POST /ledger/query
```

**Request Body:**
```json
{
  "filters": {
    "type": "payment",
    "status": "success",
    "from_date": "2025-01-01T00:00:00Z",
    "to_date": "2025-12-31T23:59:59Z"
  },
  "page": 1,
  "limit": 50
}
```

#### Get Ledger Entry
Retrieve specific ledger entry.

```http
GET /ledger/entries/{entry_id}
```

### 7. Mobile Money Operator API

#### Check Balance
Query mobile money balance.

```http
POST /mobile-money/balance
```

**Request Body:**
```json
{
  "provider": "mpesa",
  "phone_number": "+254712345678"
}
```

#### Send Money
Initiate mobile money transfer.

```http
POST /mobile-money/transfer
```

**Request Body:**
```json
{
  "provider": "mpesa",
  "from_number": "+254712345678",
  "to_number": "+254787654321",
  "amount": 500.00,
  "currency": "KES"
}
```

### 8. Cryptocurrency Mining Pool API

#### Get Pool Stats
Retrieve mining pool statistics.

```http
GET /mining/pools/{pool_id}/stats
```

**Response:**
```json
{
  "success": true,
  "data": {
    "pool_id": "pool_btc_1",
    "cryptocurrency": "BTC",
    "hashrate": "1500 TH/s",
    "miners": 250,
    "blocks_found": 12,
    "last_block": "2025-12-31T04:30:00Z"
  }
}
```

#### Get Miner Stats
Get statistics for a specific miner.

```http
GET /mining/miners/{miner_id}/stats
```

### 9. Backup & Recovery API

#### Create Backup
Initiate a data backup.

```http
POST /backup/create
```

**Request Body:**
```json
{
  "resource_type": "database",
  "resource_id": "db_prod_123",
  "backup_type": "full",
  "encryption": true
}
```

#### List Backups
Retrieve available backups.

```http
GET /backup/list?resource_id=db_prod_123
```

#### Restore Backup
Restore data from backup.

```http
POST /backup/{backup_id}/restore
```

## Webhooks

### Configuring Webhooks
Set up webhook endpoints to receive real-time notifications:

```http
POST /webhooks/configure
```

**Request Body:**
```json
{
  "url": "https://yoursite.com/webhooks/zcombinator",
  "events": ["payment.success", "order.created", "refund.completed"],
  "secret": "your_webhook_secret"
}
```

### Webhook Events

#### payment.success
```json
{
  "event": "payment.success",
  "timestamp": "2025-12-31T05:00:00Z",
  "data": {
    "transaction_id": "txn_abc123xyz",
    "amount": 1000.00,
    "currency": "KES",
    "reference": "ORDER-123456"
  }
}
```

#### order.created
```json
{
  "event": "order.created",
  "timestamp": "2025-12-31T05:00:00Z",
  "data": {
    "order_id": "ord_xyz789",
    "total": 25000.00,
    "status": "pending"
  }
}
```

### Verifying Webhooks
Verify webhook signatures using HMAC SHA-256:

```python
import hmac
import hashlib

def verify_webhook(payload, signature, secret):
    computed = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(computed, signature)
```

## SDKs & Libraries

### Python
```bash
pip install zcombinator-sdk
```

```python
from zcombinator import ZCombinatorClient

client = ZCombinatorClient(api_key='your_api_key')

# Initialize payment
payment = client.payments.initialize(
    amount=1000.00,
    currency='KES',
    payment_method='mobile_money',
    phone_number='+254712345678'
)
```

### JavaScript/Node.js
```bash
npm install @zcombinator/sdk
```

```javascript
const ZCombinator = require('@zcombinator/sdk');

const client = new ZCombinator({ apiKey: 'your_api_key' });

// Initialize payment
const payment = await client.payments.initialize({
  amount: 1000.00,
  currency: 'KES',
  paymentMethod: 'mobile_money',
  phoneNumber: '+254712345678'
});
```

### oloyelang
```oloyelang
import zcombinator

client = zcombinator.Client(api_key: "your_api_key")

payment = client.payments.initialize(
  amount: 1000.00,
  currency: "KES",
  payment_method: "mobile_money",
  phone_number: "+254712345678"
)
```

## Error Codes

| Code | Description |
|------|-------------|
| `INVALID_REQUEST` | Request parameters are invalid |
| `AUTHENTICATION_FAILED` | Invalid API key or token |
| `INSUFFICIENT_FUNDS` | Insufficient balance for operation |
| `TRANSACTION_FAILED` | Payment transaction failed |
| `RESOURCE_NOT_FOUND` | Requested resource not found |
| `RATE_LIMIT_EXCEEDED` | API rate limit exceeded |
| `SERVER_ERROR` | Internal server error |
| `SERVICE_UNAVAILABLE` | Service temporarily unavailable |

## Testing

### Sandbox Environment
Use the sandbox environment for testing:
- Base URL: `https://sandbox-api.zcombinator.africa/v1`
- Test API keys available in developer dashboard
- No real money transactions
- Reset data weekly

### Test Cards (for testing)
- Success: `+254700000000`
- Insufficient funds: `+254700000001`
- Failed transaction: `+254700000002`

## Support

- **Documentation**: https://docs.zcombinator.africa
- **API Status**: https://status.zcombinator.africa
- **Developer Forum**: https://community.zcombinator.africa
- **Support Email**: api-support@zcombinator.africa
- **Emergency**: +254-XXX-XXXXXX (24/7)

## Changelog

### v1.0.0 (2025-12-31)
- Initial API release
- Payment processing endpoints
- E-commerce integration
- AI assistant API
- Mobile money integration
- Content streaming API
- Investment platform API
- Ledger query API
- Mining pool API
- Backup & recovery API

## Terms of Service

By using the Z-Combinator API, you agree to:
- Use the API only for lawful purposes
- Protect your API keys and credentials
- Comply with rate limiting policies
- Not abuse or overload the API
- Follow data protection regulations
- Maintain security best practices

## License

The Z-Combinator API is proprietary software. See full terms at https://zcombinator.africa/terms
