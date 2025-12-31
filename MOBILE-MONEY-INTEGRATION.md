# Mobile Money Operator Integration

## Overview

The Z-Combinator Africa Ecosystem provides seamless integration with major African mobile money operators, enabling millions of users to transact using their mobile money accounts. This document details the mobile money integration architecture, supported operators, and implementation guidelines.

## Supported Mobile Money Operators

### 1. M-Pesa
**Countries**: Kenya, Tanzania, Mozambique, Lesotho, Ghana, Egypt, South Africa  
**Provider**: Safaricom (Kenya), Vodacom (Tanzania)  
**Users**: 50+ million active users

**Features**:
- STK Push (SIM Toolkit Push) for payments
- Business to Customer (B2C) transfers
- Customer to Business (C2B) payments
- Balance inquiry
- Transaction status query
- Bill payment integration

### 2. MTN Mobile Money (MoMo)
**Countries**: Nigeria, Ghana, Uganda, Rwanda, Cameroon, Côte d'Ivoire, Benin, Congo, Zambia, South Africa  
**Provider**: MTN Group  
**Users**: 60+ million active users

**Features**:
- Collection API for receiving payments
- Disbursement API for sending money
- Transfer API for wallet-to-wallet
- Balance check
- Transaction verification
- Merchant payment integration

### 3. Airtel Money
**Countries**: Nigeria, Kenya, Tanzania, Uganda, Rwanda, Zambia, Malawi, Chad, Niger, Madagascar  
**Provider**: Airtel Africa  
**Users**: 30+ million active users

**Features**:
- Payment collection
- Money disbursement
- Balance inquiry
- Transaction status
- Merchant payments
- Bulk disbursement

### 4. Orange Money
**Countries**: Senegal, Côte d'Ivoire, Mali, Cameroon, Burkina Faso, Madagascar  
**Provider**: Orange Group  
**Users**: 25+ million active users

**Features**:
- Web payment API
- Payment collection
- Merchant services
- Balance check
- Transaction history

### 5. Tigo Pesa
**Countries**: Tanzania, Rwanda, Ghana  
**Provider**: Millicom  
**Users**: 10+ million active users

**Features**:
- USSD integration
- API-based payments
- Disbursement services
- Balance inquiry

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Application Layer                  │
│    (shop.africa, oloyepay checkout, mobile apps)            │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ HTTPS/REST API
                        │
┌───────────────────────▼─────────────────────────────────────┐
│              oloyepay Payment Gateway                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │        Mobile Money Integration Layer                │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐     │  │
│  │  │  M-Pesa    │  │  MTN MoMo  │  │Airtel Money│     │  │
│  │  │  Adapter   │  │  Adapter   │  │  Adapter   │     │  │
│  │  └──────┬─────┘  └──────┬─────┘  └──────┬─────┘     │  │
│  └─────────┼────────────────┼────────────────┼───────────┘  │
└────────────┼────────────────┼────────────────┼──────────────┘
             │                │                │
             │ API Calls      │ API Calls      │ API Calls
             │                │                │
┌────────────▼────────────────▼────────────────▼──────────────┐
│              Mobile Money Operator Systems                   │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐           │
│  │  M-Pesa    │  │  MTN MoMo  │  │Airtel Money│           │
│  │  Platform  │  │  Platform  │  │  Platform  │           │
│  └────────────┘  └────────────┘  └────────────┘           │
└──────────────────────────────────────────────────────────────┘
             │                │                │
             │ SMS/USSD       │ SMS/USSD       │ SMS/USSD
             │                │                │
┌────────────▼────────────────▼────────────────▼──────────────┐
│                    Customer Mobile Phones                    │
└──────────────────────────────────────────────────────────────┘
```

## Payment Flow

### Standard Payment Flow (C2B - Customer to Business)

1. **Initiation**
   - Customer selects mobile money as payment method
   - Enters phone number and amount
   - oloyepay validates input and initiates payment

2. **STK Push / USSD Prompt**
   - Payment request sent to mobile operator
   - Customer receives prompt on their phone
   - Customer enters PIN to authorize

3. **Authorization**
   - Mobile operator validates PIN and balance
   - Transaction authorized or declined
   - Response sent back to oloyepay

4. **Completion**
   - oloyepay receives confirmation
   - Transaction recorded in oloyeDB ledger
   - Customer and merchant notified
   - Goods/services released

5. **Settlement**
   - Funds settled to merchant account
   - Transaction fees calculated
   - Settlement report generated

### Disbursement Flow (B2C - Business to Customer)

1. **Request**
   - Merchant initiates disbursement via API
   - Specifies recipient phone and amount
   - oloyepay validates request

2. **Processing**
   - oloyepay debits merchant account
   - Sends transfer request to mobile operator
   - Operator processes transfer

3. **Confirmation**
   - Customer receives money in mobile wallet
   - SMS notification sent to customer
   - oloyepay receives confirmation
   - Transaction recorded in ledger

## Implementation Guide

### M-Pesa Integration (Kenya Example)

#### Prerequisites
- M-Pesa Business Account (Paybill or Till Number)
- API credentials from Safaricom Daraja Portal
- Consumer Key and Consumer Secret
- Passkey for STK Push
- Callback URL for receiving responses

#### STK Push Implementation

```python
import requests
import base64
from datetime import datetime

class MPesaIntegration:
    def __init__(self, consumer_key, consumer_secret, passkey, shortcode):
        self.consumer_key = consumer_key
        self.consumer_secret = consumer_secret
        self.passkey = passkey
        self.shortcode = shortcode
        self.base_url = "https://api.safaricom.co.ke"
    
    def get_access_token(self):
        """Get OAuth access token"""
        url = f"{self.base_url}/oauth/v1/generate?grant_type=client_credentials"
        auth = base64.b64encode(
            f"{self.consumer_key}:{self.consumer_secret}".encode()
        ).decode()
        
        headers = {"Authorization": f"Basic {auth}"}
        response = requests.get(url, headers=headers)
        return response.json()["access_token"]
    
    def stk_push(self, phone_number, amount, account_reference, callback_url):
        """Initiate STK Push payment"""
        access_token = self.get_access_token()
        timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
        
        # Generate password
        password_str = f"{self.shortcode}{self.passkey}{timestamp}"
        password = base64.b64encode(password_str.encode()).decode()
        
        url = f"{self.base_url}/mpesa/stkpush/v1/processrequest"
        headers = {"Authorization": f"Bearer {access_token}"}
        
        payload = {
            "BusinessShortCode": self.shortcode,
            "Password": password,
            "Timestamp": timestamp,
            "TransactionType": "CustomerPayBillOnline",
            "Amount": int(amount),
            "PartyA": phone_number,
            "PartyB": self.shortcode,
            "PhoneNumber": phone_number,
            "CallBackURL": callback_url,
            "AccountReference": account_reference,
            "TransactionDesc": "Payment"
        }
        
        response = requests.post(url, json=payload, headers=headers)
        return response.json()
    
    def query_transaction(self, checkout_request_id):
        """Query STK Push transaction status"""
        access_token = self.get_access_token()
        timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
        
        password_str = f"{self.shortcode}{self.passkey}{timestamp}"
        password = base64.b64encode(password_str.encode()).decode()
        
        url = f"{self.base_url}/mpesa/stkpushquery/v1/query"
        headers = {"Authorization": f"Bearer {access_token}"}
        
        payload = {
            "BusinessShortCode": self.shortcode,
            "Password": password,
            "Timestamp": timestamp,
            "CheckoutRequestID": checkout_request_id
        }
        
        response = requests.post(url, json=payload, headers=headers)
        return response.json()
```

### MTN Mobile Money Integration

```python
import requests
import uuid

class MTNMoMoIntegration:
    def __init__(self, subscription_key, api_user, api_key, environment="sandbox"):
        self.subscription_key = subscription_key
        self.api_user = api_user
        self.api_key = api_key
        self.environment = environment
        self.base_url = f"https://{environment}.momodeveloper.mtn.com"
    
    def create_access_token(self):
        """Create OAuth access token"""
        url = f"{self.base_url}/collection/token/"
        headers = {
            "Ocp-Apim-Subscription-Key": self.subscription_key,
            "Authorization": f"Basic {self.get_basic_auth()}"
        }
        
        response = requests.post(url, headers=headers)
        return response.json()["access_token"]
    
    def request_to_pay(self, phone_number, amount, currency, reference):
        """Request payment from customer"""
        access_token = self.create_access_token()
        transaction_id = str(uuid.uuid4())
        
        url = f"{self.base_url}/collection/v1_0/requesttopay"
        headers = {
            "Authorization": f"Bearer {access_token}",
            "X-Reference-Id": transaction_id,
            "X-Target-Environment": self.environment,
            "Ocp-Apim-Subscription-Key": self.subscription_key,
            "Content-Type": "application/json"
        }
        
        payload = {
            "amount": str(amount),
            "currency": currency,
            "externalId": reference,
            "payer": {
                "partyIdType": "MSISDN",
                "partyId": phone_number
            },
            "payerMessage": "Payment request",
            "payeeNote": "Payment received"
        }
        
        response = requests.post(url, json=payload, headers=headers)
        return {"transaction_id": transaction_id, "status": response.status_code}
    
    def get_transaction_status(self, transaction_id):
        """Check transaction status"""
        access_token = self.create_access_token()
        
        url = f"{self.base_url}/collection/v1_0/requesttopay/{transaction_id}"
        headers = {
            "Authorization": f"Bearer {access_token}",
            "X-Target-Environment": self.environment,
            "Ocp-Apim-Subscription-Key": self.subscription_key
        }
        
        response = requests.get(url, headers=headers)
        return response.json()
```

## USSD Integration

For feature phone users without smartphones, USSD provides accessible payment:

### USSD Flow
1. Customer dials USSD code (e.g., `*123*456#`)
2. Merchant code and amount prompted
3. Customer confirms transaction
4. PIN entry for authorization
5. Transaction processed
6. Confirmation SMS sent

### USSD Menu Example
```
*123*456#
1. Pay Merchant
2. Check Balance
3. Transaction History

> Enter merchant code: 789012
> Enter amount: 1000
> Confirm payment of KES 1000 to Shop.Africa?
  1. Yes
  2. No

> Enter PIN: ****
> Payment successful! 
  Receipt: MPESA123XYZ
  Amount: KES 1000
  Balance: KES 5000
```

## Security Best Practices

### 1. Credential Management
- Store API keys in secure vault (never in code)
- Use environment variables for configuration
- Rotate credentials regularly
- Implement IP whitelisting

### 2. Transaction Security
- Validate all phone numbers before processing
- Implement amount limits and thresholds
- Monitor for suspicious patterns
- Enable fraud detection algorithms

### 3. Data Protection
- Encrypt sensitive data in transit and at rest
- Never log customer PINs or passwords
- Comply with PCI-DSS standards
- Implement data retention policies

### 4. API Security
- Use HTTPS for all API calls
- Implement request signing
- Validate webhook signatures
- Rate limit API requests

## Error Handling

### Common Errors

| Error Code | Description | Action |
|------------|-------------|---------|
| `INSUFFICIENT_FUNDS` | Customer balance too low | Request lower amount |
| `INVALID_PHONE_NUMBER` | Phone number format invalid | Validate input |
| `TRANSACTION_TIMEOUT` | Request timed out | Query status and retry |
| `DUPLICATE_TRANSACTION` | Transaction already processed | Check ledger |
| `ACCOUNT_BLOCKED` | Customer account blocked | Contact operator |
| `SERVICE_UNAVAILABLE` | Operator system down | Retry later |

### Retry Logic

```python
import time

def process_payment_with_retry(payment_func, max_retries=3):
    """Process payment with exponential backoff retry"""
    for attempt in range(max_retries):
        try:
            result = payment_func()
            if result['status'] == 'success':
                return result
            
            if result['error'] in ['SERVICE_UNAVAILABLE', 'TIMEOUT']:
                wait_time = 2 ** attempt  # Exponential backoff
                time.sleep(wait_time)
                continue
            else:
                return result  # Non-retryable error
                
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)
    
    return {"status": "failed", "error": "MAX_RETRIES_EXCEEDED"}
```

## Testing

### Sandbox Numbers
Each operator provides test phone numbers:

**M-Pesa Sandbox**
- Success: 254708374149
- Insufficient funds: 254711111111
- Invalid account: 254722222222

**MTN MoMo Sandbox**
- Success: 46733123453
- Rejected: 46733123454
- Timeout: 46733123455

**Airtel Sandbox**
- Success: 256700000000
- Failed: 256700000001

## Monitoring & Analytics

### Key Metrics
- Transaction success rate
- Average processing time
- Failed transaction reasons
- Operator availability
- Revenue by operator
- Transaction volume trends

### Alerts
- Success rate drops below 95%
- Response time exceeds 5 seconds
- Operator API unavailable
- Unusual transaction patterns
- High failure rate for specific operator

## Fees & Pricing

### Transaction Fees (Example)

| Operator | Transaction Fee | Settlement Time |
|----------|----------------|-----------------|
| M-Pesa | 1.5% + KES 5 | T+1 day |
| MTN MoMo | 2.0% | T+1 day |
| Airtel Money | 1.8% | T+2 days |
| Orange Money | 2.2% | T+2 days |

### oloyepay Commission
- E-commerce payments: 1.0%
- Peer-to-peer: 0.5%
- Bulk disbursements: 0.3%
- Enterprise volume pricing available

## Compliance & Regulations

### KYC Requirements
- Customer name verification
- Phone number ownership confirmation
- Transaction limit enforcement
- AML (Anti-Money Laundering) checks

### Regulatory Compliance
- Central Bank regulations adherence
- Payment Service Provider licensing
- Data protection laws (GDPR, local)
- Consumer protection requirements

## Support & Resources

### Developer Resources
- **M-Pesa**: https://developer.safaricom.co.ke
- **MTN MoMo**: https://momodeveloper.mtn.com
- **Airtel Money**: https://developers.airtel.africa
- **Orange Money**: https://developer.orange.com

### oloyepay Support
- **Documentation**: https://docs.oloyepay.africa/mobile-money
- **API Status**: https://status.oloyepay.africa
- **Technical Support**: mobile-money@oloyepay.africa
- **Integration Help**: +254-XXX-XXXXXX

## Roadmap

### Q1 2026
- Add support for Vodacom M-Pesa (South Africa)
- Implement QR code payments
- Enhanced fraud detection

### Q2 2026
- Cross-border mobile money transfers
- Multi-currency wallet support
- Real-time settlement options

### Q3 2026
- Integration with additional operators
- Merchant app for payment acceptance
- Advanced analytics dashboard

### Q4 2026
- Offline payment capabilities
- Biometric authentication support
- AI-powered transaction routing
