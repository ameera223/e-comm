# # 🛒 E-commerce Database Schema (PostgreSQL)

This project implements a complete E-commerce database schema using PostgreSQL. It includes relationships, audit fields, soft delete logic, and views for real-world usage such as order summaries and product availability.

---

## 📦 Tables and Relationships

### Tables:
- `categories`: Product categories
- `products`: Items for sale
- `customers`: Users who place orders
- `orders`: Purchase records
- `order_items`: Line items within each order

### Relationships:
- `products.category_id` → `categories.id`
- `orders.customer_id` → `customers.id`
- `order_items.order_id` → `orders.id`
- `order_items.product_id` → `products.id`

---

## 🧱 Full Database Schema
-- Table: categories
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Table: products
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    description TEXT,
    price NUMERIC(10,2) CHECK (price >= 0),
    stock_quantity INT CHECK (stock_quantity >= 0),
    category_id INT REFERENCES categories(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Table: customers
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(150) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    address TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Table: orders
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL REFERENCES customers(id),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'Pending',
    total_amount NUMERIC(12,2) CHECK (total_amount >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Table: order_items
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT NOT NULL REFERENCES products(id),
    quantity INT NOT NULL CHECK (quantity > 0),
    price_per_unit NUMERIC(10,2) CHECK (price_per_unit >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

📄 Sample Data

-- Categories
INSERT INTO categories (name, description) VALUES
('Electronics', 'Devices and gadgets'),
('Clothing', 'Apparel and accessories'),
('Books', 'All kinds of books');

-- Products
INSERT INTO products (name, description, price, stock_quantity, category_id) VALUES
('Smartphone', 'Latest Android smartphone', 29999.99, 50, 1),
('Laptop', 'High-performance laptop', 74999.00, 20, 1),
('T-Shirt', '100% cotton t-shirt', 499.00, 100, 2),
('Jeans', 'Slim fit jeans', 1199.00, 60, 2),
('Novel', 'Fictional novel', 299.00, 200, 3),
('Textbook', 'Academic textbook', 799.00, 150, 3);

-- Customers
INSERT INTO customers (full_name, email, phone, address) VALUES
('Alice Johnson', 'alice@example.com', '9876543210', '123 Maple Street, NY'),
('Bob Smith', 'bob@example.com', '9876500000', '456 Oak Avenue, LA'),
('Charlie Brown', 'charlie@example.com', '9999912345', '789 Pine Road, TX');

-- Orders
INSERT INTO orders (customer_id, status, total_amount) VALUES
(1, 'Pending', 30598.99),
(2, 'Shipped', 499.00),
(3, 'Delivered', 1998.00);

-- Order Items
INSERT INTO order_items (order_id, product_id, quantity, price_per_unit) VALUES
(1, 1, 1, 29999.99),
(1, 5, 2, 299.00),
(2, 3, 1, 499.00),
(3, 4, 1, 1199.00),
(3, 6, 1, 799.00);

👁️‍🗨️ Views
sql
Copy
Edit
-- View: Order Summary
CREATE VIEW order_summary_view AS
SELECT 
    o.id AS order_id,
    o.order_date,
    c.full_name AS customer_name,
    c.email,
    o.status,
    o.total_amount,
    COUNT(oi.id) AS total_items
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
WHERE o.is_deleted = FALSE AND oi.is_deleted = FALSE
GROUP BY o.id, o.order_date, c.full_name, c.email, o.status, o.total_amount;

-- View: Product Availability
CREATE VIEW product_availability_view AS
SELECT 
    p.id AS product_id,
    p.name AS product_name,
    c.name AS category_name,
    p.price,
    p.stock_quantity,
    CASE 
        WHEN p.stock_quantity > 0 THEN 'In Stock'
        ELSE 'Out of Stock'
    END AS availability
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.is_deleted = FALSE AND c.is_deleted = FALSE;

✅ Sample Queries

-- Get all non-deleted products
SELECT * FROM products WHERE is_deleted = FALSE;

-- Get current stock availability
SELECT * FROM product_availability_view;

-- Get summary of all orders
SELECT * FROM order_summary_view;

🗃️ Soft Delete Example
-- Soft delete a product
UPDATE products SET is_deleted = TRUE WHERE id = 1;
Views automatically exclude soft-deleted records.

📜 License
MIT — free to use and modify for any purpose.
