# Database Management System 🗄️

A comprehensive **full-stack database project** demonstrating professional database design, advanced SQL queries, relational modeling, and backend integration. Features real-world scenarios with complete documentation.

---

## 🌟 Project Highlights

✅ **Database Design**
- ER (Entity-Relationship) Diagrams
- Database Normalization (1NF to 3NF)
- Proper indexing strategies
- Constraint implementation

✅ **Advanced SQL**
- Complex queries with JOINs
- Aggregate functions & GROUP BY
- Subqueries & CTEs
- Views & Stored Procedures
- Transactions & ACID properties

✅ **Backend Integration**
- Python/Node.js API
- RESTful endpoints
- Database connection pooling
- Error handling & validation

✅ **Documentation**
- Schema diagrams
- Query explanations
- Setup guides
- Performance analysis

---

## 📋 Database Schema

### Entity-Relationship Diagram

```
┌─────────────┐         ┌─────────────┐
│   Users     │         │   Orders    │
├─────────────┤         ├─────────────┤
│ user_id (PK)├────────→│ order_id(PK)│
│ username    │   1:N   │ user_id(FK) │
│ email       │         │ order_date  │
│ created_at  │         │ total_price │
└─────────────┘         └─────────────┘
                                │
                                │ 1:N
                                ↓
                        ┌─────────────────┐
                        │  Order_Items    │
                        ├─────────────────┤
                        │order_item_id(PK)│
                        │ order_id (FK)   │
                        │ product_id(FK)  │
                        │ quantity        │
                        │ price           │
                        └─────────────────┘
                                ↑
                                │ N:1
                                │
                        ┌─────────────────┐
                        │   Products      │
                        ├─────────────────┤
                        │product_id(PK)   │
                        │ name            │
                        │ price           │
                        │ category        │
                        │ stock           │
                        └─────────────────┘
```

### Table Definitions

**Users Table**
```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_users_email ON users(email);
```

**Products Table**
```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    category VARCHAR(50) NOT NULL,
    stock INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT price_positive CHECK (price > 0),
    CONSTRAINT stock_non_negative CHECK (stock >= 0)
);

CREATE INDEX idx_products_category ON products(category);
```

**Orders Table**
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_price DECIMAL(10, 2),
    status VARCHAR(20) DEFAULT 'pending',
    
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

CREATE INDEX idx_orders_user ON orders(user_id);
```

**Order_Items Table**
```sql
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    
    CONSTRAINT quantity_positive CHECK (quantity > 0)
);
```

---

## 🔍 Advanced SQL Queries

### Query 1: Top Customers by Spending
```sql
SELECT 
    u.user_id,
    u.username,
    COUNT(DISTINCT o.order_id) as total_orders,
    SUM(o.total_price) as total_spent,
    AVG(o.total_price) as avg_order_value
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE o.order_date >= DATE_SUB(NOW(), INTERVAL 1 YEAR)
GROUP BY u.user_id, u.username
HAVING total_spent > 1000
ORDER BY total_spent DESC
LIMIT 10;
```

### Query 2: Product Performance Analysis
```sql
SELECT 
    p.product_id,
    p.name,
    COUNT(oi.order_item_id) as times_ordered,
    SUM(oi.quantity) as total_quantity_sold,
    SUM(oi.quantity * oi.price) as total_revenue,
    AVG(oi.quantity) as avg_quantity_per_order,
    RANK() OVER (ORDER BY SUM(oi.quantity * oi.price) DESC) as revenue_rank
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE MONTH(oi.order_date) = MONTH(CURDATE())
GROUP BY p.product_id, p.name
ORDER BY total_revenue DESC;
```

### Query 3: Customer Purchase History with Ranking
```sql
SELECT 
    u.username,
    o.order_id,
    o.order_date,
    o.total_price,
    ROW_NUMBER() OVER (PARTITION BY u.user_id ORDER BY o.order_date DESC) as order_rank,
    LAG(o.total_price) OVER (PARTITION BY u.user_id ORDER BY o.order_date) as previous_order_value
FROM users u
INNER JOIN orders o ON u.user_id = o.user_id
ORDER BY u.user_id, o.order_date DESC;
```

### Query 4: CTE for Hierarchical Analysis
```sql
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', o.order_date) as month,
        SUM(o.total_price) as monthly_revenue,
        COUNT(*) as order_count
    FROM orders o
    GROUP BY DATE_TRUNC('month', o.order_date)
),
sales_with_growth AS (
    SELECT 
        month,
        monthly_revenue,
        order_count,
        LAG(monthly_revenue) OVER (ORDER BY month) as prev_revenue,
        ROUND(((monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY month)) / 
               LAG(monthly_revenue) OVER (ORDER BY month) * 100), 2) as growth_percent
    FROM monthly_sales
)
SELECT * FROM sales_with_growth
WHERE growth_percent IS NOT NULL;
```

---

## 📂 Project Structure

```
database-management-system/
├── schema/
│   ├── tables.sql           # Table definitions
│   ├── indexes.sql          # Index creation
│   ├── constraints.sql      # Constraints & triggers
│   ├── er_diagram.md        # ER diagram documentation
│   └── normalization.md     # Normalization explanation
├── queries/
│   ├── basic/
│   │   ├── select_queries.sql
│   │   ├── join_queries.sql
│   │   └── aggregate_queries.sql
│   ├── advanced/
│   │   ├── window_functions.sql
│   │   ├── cte_queries.sql
│   │   └── subqueries.sql
│   └── reports/
│       ├── sales_report.sql
│       ├── customer_analysis.sql
│       └── inventory_report.sql
├── data/
│   ├── sample_data.sql      # Sample data for testing
│   └── backup/              # Database backups
├── backend/
│   ├── app.py               # Flask/FastAPI app
│   ├── models.py            # SQLAlchemy models
│   ├── routes.py            # API endpoints
│   └── database.py          # DB connection
├── scripts/
│   ├── setup_db.sh          # Setup script
│   ├── backup_db.sh         # Backup script
│   └── restore_db.sh        # Restore script
├── tests/
│   ├── test_queries.py
│   ├── test_api.py
│   └── test_constraints.py
└── documentation/
    ├── README.md
    ├── SETUP.md
    ├── QUERIES.md
    └── API_DOCS.md
```

---

## 🚀 Setup & Installation

### Prerequisites
- PostgreSQL 12+
- Python 3.8+
- Node.js (optional, for Express backend)

### Database Setup

```bash
# 1. Create database
createdb e_commerce_db

# 2. Connect to database
psql -U postgres -d e_commerce_db

# 3. Run schema setup
\i schema/tables.sql
\i schema/indexes.sql
\i schema/constraints.sql

# 4. Load sample data
\i data/sample_data.sql

# 5. Verify setup
\dt  -- List all tables
```

### Backend Setup (Python)

```bash
# Install dependencies
pip install flask sqlalchemy psycopg2-binary

# Run application
python3 app.py

# API runs on http://localhost:5000
```

---

## 📊 Database Normalization

### 1NF (First Normal Form)
- ✅ All attributes contain atomic values
- ✅ No repeating groups
- ✅ Primary key defined

### 2NF (Second Normal Form)
- ✅ In 1NF
- ✅ All non-key attributes depend on entire primary key
- ✅ No partial dependencies

### 3NF (Third Normal Form)
- ✅ In 2NF
- ✅ No transitive dependencies
- ✅ Non-key attributes depend only on primary key

---

## 🔗 API Endpoints

### Users
```
GET    /api/users              # List all users
POST   /api/users              # Create user
GET    /api/users/{id}         # Get user details
PUT    /api/users/{id}         # Update user
DELETE /api/users/{id}         # Delete user
```

### Products
```
GET    /api/products           # List products
POST   /api/products           # Add product
GET    /api/products/{id}      # Get product
PUT    /api/products/{id}      # Update product
DELETE /api/products/{id}      # Delete product
```

### Orders
```
GET    /api/orders             # List orders
POST   /api/orders             # Create order
GET    /api/orders/{id}        # Get order details
PUT    /api/orders/{id}        # Update order
DELETE /api/orders/{id}        # Cancel order
```

---

## 🧪 Testing

```bash
# Run all tests
python3 -m pytest tests/ -v

# Test specific module
python3 -m pytest tests/test_queries.py

# Test with coverage
pytest --cov=backend tests/

# Test API endpoints
pytest tests/test_api.py -v
```

---

## 📈 Performance Optimization

### Indexes Strategy
```
✅ User lookups: Index on email
✅ Order lookups: Index on user_id
✅ Product search: Index on category & name
✅ Timestamp queries: Index on created_at
```

### Query Optimization Tips
1. Use EXPLAIN to analyze queries
2. Avoid SELECT *
3. Use appropriate JOINs
4. Create composite indexes for common filters
5. Partition large tables

---

## 📚 Learning Outcomes

By working through this project, you'll understand:

- ✅ Database design principles
- ✅ ER modeling & relationships
- ✅ SQL normalization
- ✅ Complex query writing
- ✅ Index optimization
- ✅ Transaction management
- ✅ Backend API integration
- ✅ Data validation & constraints

---

## 🐛 Common Queries

### Calculate Order Statistics
```sql
SELECT 
    DATE(order_date) as date,
    COUNT(*) as total_orders,
    SUM(total_price) as daily_revenue,
    AVG(total_price) as avg_order_value
FROM orders
GROUP BY DATE(order_date)
ORDER BY date DESC;
```

### Find Low Stock Products
```sql
SELECT 
    product_id,
    name,
    stock,
    CASE 
        WHEN stock < 10 THEN 'Critical'
        WHEN stock < 50 THEN 'Low'
        ELSE 'Adequate'
    END as stock_status
FROM products
WHERE stock < 50
ORDER BY stock ASC;
```

---

## 🤝 Contributing

Suggestions for improvement:
- [ ] Add user authentication system
- [ ] Implement caching layer
- [ ] Add data analytics dashboard
- [ ] Implement audit logging
- [ ] Add data validation layer

---

## 📄 License

MIT License - Learn and practice freely!

---

## 👤 Author

**Mahdi Kammoun**  
Computer Science Student | IIT Sfax (Génie Logiciel et Systèmes d'Information - GLSI)  

📧 Contact: **[mkammoun75@gmail.com](mailto:mkammoun75@gmail.com)**  

*Last Updated: November 2025*  

⭐ If this helps you learn, please star the repository!
