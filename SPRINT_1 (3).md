# SPRINT 1 — System Architecture & Scope

## Project Title
**SmartCart: AI-Assisted Consumer Electronics E-Commerce Platform**

## Course
E-Commerce

---

## 1. Target Audience & Market Focus

### Primary Persona
The primary users are retail consumers in Pakistan who want to purchase consumer electronics online. The platform is designed for university students, working professionals, and small households looking for reliable products, transparent pricing, product comparisons, and convenient home delivery.

### Core Pain Point
Customers often struggle with unreliable product information, unclear stock availability, difficult product searching, hidden costs, and a lack of trust in online electronics sellers. SmartCart addresses these problems through organized product categories, search and filtering, real-time inventory visibility, secure authentication, cart management, and a streamlined checkout process.

### Domain Scope
The platform focuses on the **Consumer Electronics** market, including:

- Smartphones and accessories
- Laptops and computer peripherals
- Headphones and audio devices
- Smartwatches
- Home networking devices
- Chargers, cables, and other electronic accessories

The MVP will focus on product discovery, shopping-cart functionality, order placement, and basic administrator inventory management.

---

## 2. Minimum Viable Product (MVP) Feature Scope

| Category | Feature Name | Description | Priority |
|---|---|---|---|
| Authentication | User Registration & Login | Account creation, secure password hashing, login, logout, and JWT-based authentication. | High (MVP) |
| Catalog | Product List & Search | Browse products with keyword search, category filtering, sorting, and product details. | High (MVP) |
| Cart | Cart Management | Add products, update quantities, remove items, and maintain a persistent shopping cart. | High (MVP) |
| Checkout | Order Processing | Validate cart items, calculate total amount, create an order, and support mock payment processing. | High (MVP) |
| Orders | Order History & Status | Allow authenticated users to view previous orders and their order statuses. | Medium (MVP) |
| Admin | Inventory Control | Admin users can create, update, delete, and manage product stock and categories. | Medium (MVP) |

### MVP Boundaries

The following features are outside the Sprint 1/MVP scope:

- Advanced AI product recommendations
- Real payment settlement and banking integration
- Multi-vendor seller management
- Delivery partner integration
- Loyalty programs
- Live chat support
- Advanced analytics dashboards

---

## 3. Tech Stack Selection & Justification

### Frontend Framework: Next.js

**Justification:** Next.js is selected because it provides a structured React-based framework with routing, reusable components, strong performance, and support for server-side rendering. Compared with a plain React setup, Next.js offers better project organization and built-in optimization while remaining suitable for a semester-level e-commerce project.

### Backend Infrastructure: Node.js with Express.js

**Justification:** Node.js and Express.js provide a lightweight, scalable, and widely supported backend environment. They are selected over heavier frameworks because they allow rapid REST API development, have a large ecosystem, and use JavaScript across both frontend and backend, reducing development complexity.

### Database Management System: PostgreSQL

**Justification:** PostgreSQL is selected because the application requires strong relational integrity between users, products, carts, orders, and order items. Compared with MongoDB, PostgreSQL provides reliable foreign-key constraints, transactions, normalization, and structured SQL querying, which are important for order and payment-related data.

### Caching & Asynchronous Processing: Redis (Optional)

**Justification:** Redis may be introduced for caching frequently accessed product data, temporary sessions, rate limiting, and future background jobs. It is optional for the MVP because the initial system can operate using PostgreSQL and the Express backend without introducing unnecessary infrastructure complexity.

### Proposed Architecture

```text
Client Browser
     |
     v
Next.js Frontend
     |
     v
Express.js REST API
     |
     +------------------+
     |                  |
     v                  v
PostgreSQL           Redis (Optional)
Database             Cache / Sessions
```

---

## 4. Entity-Relationship Diagram (ERD)

### Mermaid ERD

```mermaid
erDiagram
    USERS ||--o{ ORDERS : places
    USERS ||--o| CARTS : owns
    CARTS ||--o{ CART_ITEMS : contains
    PRODUCTS ||--o{ CART_ITEMS : added_to
    CATEGORIES ||--o{ PRODUCTS : categorizes
    ORDERS ||--|{ ORDER_ITEMS : contains
    PRODUCTS ||--o{ ORDER_ITEMS : included_in

    USERS {
        INTEGER id PK
        VARCHAR full_name
        VARCHAR email UK
        VARCHAR password_hash
        VARCHAR role
        TIMESTAMP created_at
    }

    CATEGORIES {
        INTEGER id PK
        VARCHAR name UK
        VARCHAR description
    }

    PRODUCTS {
        INTEGER id PK
        INTEGER category_id FK
        VARCHAR name
        TEXT description
        DECIMAL price
        INTEGER stock_quantity
        VARCHAR image_url
        BOOLEAN is_active
        TIMESTAMP created_at
    }

    CARTS {
        INTEGER id PK
        INTEGER user_id FK UK
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    CART_ITEMS {
        INTEGER id PK
        INTEGER cart_id FK
        INTEGER product_id FK
        INTEGER quantity
        DECIMAL unit_price
    }

    ORDERS {
        INTEGER id PK
        INTEGER user_id FK
        DECIMAL total_amount
        VARCHAR status
        VARCHAR payment_status
        TIMESTAMP ordered_at
    }

    ORDER_ITEMS {
        INTEGER id PK
        INTEGER order_id FK
        INTEGER product_id FK
        INTEGER quantity
        DECIMAL unit_price
    }
```

### Entity and Attribute Specifications

#### USERS
- `id INTEGER PRIMARY KEY`
- `full_name VARCHAR(100) NOT NULL`
- `email VARCHAR(150) UNIQUE NOT NULL`
- `password_hash VARCHAR(255) NOT NULL`
- `role VARCHAR(20) DEFAULT 'customer'`
- `created_at TIMESTAMP NOT NULL`

#### CATEGORIES
- `id INTEGER PRIMARY KEY`
- `name VARCHAR(100) UNIQUE NOT NULL`
- `description VARCHAR(255)`

#### PRODUCTS
- `id INTEGER PRIMARY KEY`
- `category_id INTEGER FOREIGN KEY REFERENCES categories(id)`
- `name VARCHAR(150) NOT NULL`
- `description TEXT`
- `price DECIMAL(10,2) NOT NULL`
- `stock_quantity INTEGER NOT NULL`
- `image_url VARCHAR(500)`
- `is_active BOOLEAN DEFAULT TRUE`
- `created_at TIMESTAMP NOT NULL`

#### CARTS
- `id INTEGER PRIMARY KEY`
- `user_id INTEGER UNIQUE FOREIGN KEY REFERENCES users(id)`
- `created_at TIMESTAMP NOT NULL`
- `updated_at TIMESTAMP NOT NULL`

#### CART_ITEMS
- `id INTEGER PRIMARY KEY`
- `cart_id INTEGER FOREIGN KEY REFERENCES carts(id)`
- `product_id INTEGER FOREIGN KEY REFERENCES products(id)`
- `quantity INTEGER NOT NULL`
- `unit_price DECIMAL(10,2) NOT NULL`

#### ORDERS
- `id INTEGER PRIMARY KEY`
- `user_id INTEGER FOREIGN KEY REFERENCES users(id)`
- `total_amount DECIMAL(10,2) NOT NULL`
- `status VARCHAR(30) DEFAULT 'pending'`
- `payment_status VARCHAR(30) DEFAULT 'unpaid'`
- `ordered_at TIMESTAMP NOT NULL`

#### ORDER_ITEMS
- `id INTEGER PRIMARY KEY`
- `order_id INTEGER FOREIGN KEY REFERENCES orders(id)`
- `product_id INTEGER FOREIGN KEY REFERENCES products(id)`
- `quantity INTEGER NOT NULL`
- `unit_price DECIMAL(10,2) NOT NULL`

### Relationship Cardinality

- One user can place many orders: **USERS 1:N ORDERS**
- One user can own zero or one cart: **USERS 1:0..1 CARTS**
- One cart can contain many cart items: **CARTS 1:N CART_ITEMS**
- One product can appear in many cart items: **PRODUCTS 1:N CART_ITEMS**
- One category can contain many products: **CATEGORIES 1:N PRODUCTS**
- One order must contain one or more order items: **ORDERS 1:N ORDER_ITEMS**
- One product can appear in many order items: **PRODUCTS 1:N ORDER_ITEMS**

The many-to-many relationship between orders and products is resolved through the associative entity `ORDER_ITEMS`.

---

## Conclusion

SmartCart provides a practical and achievable architecture for a consumer-electronics e-commerce platform. The proposed MVP focuses on the essential customer journey—from registration and product discovery to cart management and order placement—while PostgreSQL ensures reliable relational data modeling and consistency. The scope is intentionally limited to features that can realistically be implemented within an academic semester.
