# Fetch.ai Merchandise Shopping Agent Workflow

## Overview
An intelligent AI agent that handles the complete shopping experience for Fetch.ai merchandise (T-shirts) including product selection, order placement, payment processing, and order tracking.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SHOPPING PHASE                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  USER QUERY: "I want to buy Fetch.ai T-shirt"                                  │
│           ↓                                                                    │
│  AGENT RESPONSE: T-shirt photos + sizes + prices                               │
│           ↓                                                                    │
│  USER SELECTION: "Select T-shirt #2 with size M"                              │
│           ↓                                                                    │
│  AGENT REQUEST: "Please provide details:"                                     │
│           ↓                                                                    │
│  USER DETAILS: Name + Address + Mobile + Password                             │
│           ↓                                                                    │
│  SUPABASE: Save order + Generate Order ID + Encrypt password                  │
│           ↓                                                                    │
│  RAZORPAY: Payment link sent to user                                          │
│           ↓                                                                    │
│  USER PAYMENT: Complete payment                                               │
│           ↓                                                                    │
│  SUPABASE: Update with payment details + status "Paid"                        │
│           ↓                                                                    │
│  CONFIRMATION: Email sent + Tracking ID provided                             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                           TRACKING PHASE                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  USER LOGIN: Mobile Number + Password                                           │
│           ↓                                                                    │
│  AGENT AUTH: "Welcome back, [Name]!"                                           │
│           ↓                                                                    │
│  USER REQUEST: "Track my order" + Tracking ID                                  │
│           ↓                                                                    │
│  AGENT RESPONSE: Order status + delivery details + progress                    │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Complete Flow States

```
[START] → [PRODUCT_SHOW] → [PRODUCT_SELECT] → [DETAILS_COLLECT] → [ORDER_SAVE] 
    ↓                                                                      ↓
[PAYMENT_LINK] ← [PAYMENT_PROCESS] ← [SUPABASE_UPDATE] ← [CONFIRMATION] 
    ↓                                                                      ↓
[COMPLETED] → [EMAIL_SENT] → [TRACKING_ID_GENERATED] → [SHIPPING_PROCESS]
                                                              ↓
[TRACKING_LOGIN] → [AUTHENTICATE] → [TRACK_ORDER] → [STATUS_UPDATE]
```

## Complete Workflow

### Phase 1: Product Selection & Ordering

#### 1. Initial Query
```
User → Agent: "I want to buy Fetch.ai T-shirt"
```

#### 2. Product Display
```
Agent → User: 
- T-shirt photos
- Available sizes (S, M, L, XL, XXL)
- Prices for each size
- Product variants/colors
```

#### 3. Product Selection
```
User → Agent: "Select T-shirt #2" (with size preference)
Agent → User: Confirms selection with details
```

#### 4. Customer Details Collection
```
Agent → User: "Please provide your details:"
Required Information:
- Full Name
- Complete Address
- Mobile Number
- Password (for future tracking)
```

#### 5. Database Storage
```
Agent → Supabase: Save order details
- Generate unique Order ID
- Store user details (password encrypted)
- Link with selected T-shirt details
- Set status: "Pending Payment"
```

#### 6. Payment Processing
```
Agent → User: Razorpay payment link
User → Razorpay: Complete payment
Agent → Supabase: Update order with:
- Payment ID
- Transaction details
- Status: "Paid"
```

#### 7. Order Confirmation
```
Agent → User: 
- Order successful confirmation
- Unique tracking ID
- Email confirmation sent automatically
- Order summary
```

---

### Phase 2: Order Tracking

#### 1. User Authentication
```
User → Agent: 
- Mobile Number
- Password
Agent → System: Verify credentials
Agent → User: "Welcome back, [Name]!"
```

#### 2. Track Order
```
User → Agent: "Track my order" + Tracking ID
Agent → Supabase: Fetch order details
Agent → User: 
- Order Status
- Expected delivery date
- Current location
- Order history
```

---

## System Architecture

### Components
1. **AI Agent** - Main conversation handler
2. **Supabase Database** - Order storage and management
3. **Razorpay** - Payment processing
4. **Email Service** - Order confirmations
5. **Product Catalog** - T-shirt inventory and photos

### Database Schema (Supabase)

```sql
-- Orders Table
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  order_id VARCHAR(50) UNIQUE NOT NULL,
  user_name VARCHAR(100) NOT NULL,
  user_mobile VARCHAR(15) NOT NULL,
  user_address TEXT NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  product_id INTEGER NOT NULL,
  product_size VARCHAR(10) NOT NULL,
  product_price DECIMAL(10,2) NOT NULL,
  payment_id VARCHAR(100),
  payment_status VARCHAR(20),
  order_status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Products Table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  image_url VARCHAR(255),
  sizes VARCHAR(50),
  prices JSONB,
  available BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Security Features

### Password Protection
- User passwords are encrypted before storing in Supabase
- Passwords are never sent back to users in responses
- Secure authentication for order tracking

### Data Privacy
- User details stored securely in encrypted format
- Payment information handled by Razorpay's secure gateway
- No sensitive data exposed in conversation logs

---

## User Experience Flow

### Shopping Journey
1. **Discovery** → User asks about T-shirt
2. **Selection** → Agent shows products with details
3. **Customization** → User selects size and confirms
4. **Registration** → User provides details and password
5. **Payment** → Secure payment through Razorpay
6. **Confirmation** → Email confirmation and tracking ID

### Tracking Journey
1. **Login** → User enters mobile and password
2. **Authentication** → Agent welcomes user back
3. **Tracking** → User provides tracking ID
4. **Updates** → Agent shows order status and progress

---

## Technical Implementation Notes

### Agent Capabilities Required
- Natural language processing
- Product catalog integration
- User authentication
- Payment gateway integration
- Email automation
- Order tracking system
- Database management

### Integration Points
- **Supabase**: Database operations
- **Razorpay**: Payment processing
- **Email Service**: Automated confirmations
- **Product API**: Inventory management
- **Shipping API**: Tracking updates

---

## Error Handling

### Common Scenarios
1. **Product Out of Stock** → Show alternatives
2. **Payment Failure** → Retry options
3. **Invalid Credentials** → Clear error messages
4. **Network Issues** → Retry mechanisms
5. **Invalid Tracking ID** → Help and support

---

## Success Metrics

### Order Completion Rate
- Percentage of initiated orders completed
- Average time from query to payment
- Customer satisfaction scores

### Support Metrics
- Tracking system usage
- Customer service interactions
- Return/exchange handling

---

This workflow ensures a seamless shopping experience while maintaining security and providing excellent customer service throughout the order lifecycle.
