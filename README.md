# Ecommerce Management DBMS Project (MSSQL)

As a part of our University Curriculum, this project demonstrates an **E-commerce Management System** using **Microsoft SQL Server (T-SQL)**.

---

##  Pre-requisite

* Microsoft SQL Server (SSMS)

---

## 📚 Contents

* Mini World and Project Description
* Basic Structure

  * Functional Requirements
  * Entity Relation (ER) Diagram and Constraints
  * Relational Database Schema
* Implementation

  * Creating Tables
  * Inserting Data
* Queries

  * Basic Queries
  * Functions (T-SQL)
  * Triggers

---

# 🧩 1. Mini World and Description

In this modern era of online shopping, no seller wants to be left behind. Moreover, due to its simplicity, the shift from an offline selling model to an online selling model is witnessing rapid growth.

Therefore, as engineers, our job is to ease this transition for sellers. Among the many components required for an online platform, the most important is a **database system**.

Hence, in this project, we design a database where **small clothing sellers can sell their products online efficiently**.

---

# 🏗️ 2. Basic Structure

## 🔹 2.1 Functional Requirements

* A new user can register on the website
* A customer can view details of products present in the cart
* A customer can view their order history
* Admin can start a sale with discounts on products
* Customers can filter products based on attributes
* A customer can add or remove products from the cart
* A seller can unregister or stop selling products
* Sellers and customers can update their details
* Admin can view products purchased on a specific date
* Admin can view the number of products sold on a specific date
* A customer can view the total price of unpurchased items in the cart
* Admin can view customers who have not made any purchases
* Admin can calculate total profit earned from the platform

---

## 🔹 2.2 Entity Relation (ER) Diagram and Constraints


<img width="5800" height="3344" alt="new_er" src="https://github.com/user-attachments/assets/9e69a43f-54d7-402a-af08-906428f1b770" />

```md
![ER Diagram](images/er-diagram.png)
```

---

## 🔹 2.3 Relational Database Schema

<img width="2476" height="2026" alt="new_relational" src="https://github.com/user-attachments/assets/c83bd060-b843-4f17-9fae-2102f7abec4f" />


```md
![Relational Schema](images/relational-schema.png)
```



##  Implementation

### 3.1 Creating Tables

```sql
CREATE TABLE Cart (
    Cart_id VARCHAR(7) PRIMARY KEY
);

CREATE TABLE Customer (
    Customer_id VARCHAR(6) PRIMARY KEY,
    c_pass VARCHAR(10) NOT NULL,
    Name VARCHAR(20) NOT NULL,
    Address VARCHAR(20) NOT NULL,
    Pincode INT NOT NULL,
    Phone_number BIGINT NOT NULL,
    Cart_id VARCHAR(7) NOT NULL,
    FOREIGN KEY (Cart_id) REFERENCES Cart(Cart_id)
);

CREATE TABLE Seller (
    Seller_id VARCHAR(6) PRIMARY KEY,
    s_pass VARCHAR(10) NOT NULL,
    Name VARCHAR(20) NOT NULL,
    Address VARCHAR(50) NOT NULL
);

CREATE TABLE Seller_Phone_num (
    Phone_num BIGINT,
    Seller_id VARCHAR(6),
    PRIMARY KEY (Phone_num, Seller_id),
    FOREIGN KEY (Seller_id) REFERENCES Seller(Seller_id) ON DELETE CASCADE
);

CREATE TABLE Product (
    Product_id VARCHAR(7) PRIMARY KEY,
    Type VARCHAR(20) NOT NULL,
    Color VARCHAR(15) NOT NULL,
    P_Size VARCHAR(5) NOT NULL,
    Gender CHAR(1) NOT NULL,
    Commission INT NOT NULL,
    Cost DECIMAL(10,2) NOT NULL,
    Quantity INT NOT NULL,
    Seller_id VARCHAR(6),
    FOREIGN KEY (Seller_id) REFERENCES Seller(Seller_id) ON DELETE SET NULL
);

CREATE TABLE Cart_item (
    Quantity_wished INT NOT NULL,
    Date_Added DATE NOT NULL,
    Cart_id VARCHAR(7),
    Product_id VARCHAR(7),
    purchased VARCHAR(3) DEFAULT 'NO',
    PRIMARY KEY (Cart_id, Product_id),
    FOREIGN KEY (Cart_id) REFERENCES Cart(Cart_id),
    FOREIGN KEY (Product_id) REFERENCES Product(Product_id)
);

CREATE TABLE Payment (
    payment_id VARCHAR(7) PRIMARY KEY,
    payment_date DATE NOT NULL,
    Payment_type VARCHAR(10) NOT NULL,
    Customer_id VARCHAR(6),
    Cart_id VARCHAR(7),
    total_amount DECIMAL(10,2),
    FOREIGN KEY (Customer_id) REFERENCES Customer(Customer_id),
    FOREIGN KEY (Cart_id) REFERENCES Cart(Cart_id)
);
```

---

### 3.2 Inserting Values

```sql
INSERT INTO Cart VALUES ('crt1011');

INSERT INTO Customer 
VALUES ('cid100','ABCM1235','rajat','G-453',632014,9893135876,'crt1011');

INSERT INTO Seller 
VALUES ('sid100','12345','aman','delhi cmc');

INSERT INTO Product 
VALUES ('pid1001','jeans','red','32','M',10,10005,20,'sid100');

INSERT INTO Seller_Phone_num 
VALUES (9943336206,'sid100');

INSERT INTO Cart_item 
VALUES (3,'1999-10-10','crt1011','pid1001','Y');

INSERT INTO Payment 
VALUES ('pmt1001','1999-10-10','online','cid100','crt1011',NULL);
```

---

# 4. Queries

## 4.1 Basic Queries

###  If the customer wants to see details of product present in the cart

```sql
SELECT * 
FROM Product 
WHERE Product_id IN (
    SELECT Product_id 
    FROM Cart_item 
    WHERE Cart_id IN (
        SELECT Cart_id 
        FROM Customer 
        WHERE Customer_id = 'cid100'
    )
    AND purchased = 'NO'
);
```

---

###  If a customer wants to see order history

```sql
SELECT Product_id, Quantity_wished 
FROM Cart_item 
WHERE purchased = 'Y' 
AND Cart_id IN (
    SELECT Cart_id 
    FROM Customer 
    WHERE Customer_id = 'cid101'
);
```

---

###  Filter products (size, gender, type)

```sql
SELECT Product_id, Color, Cost, Seller_id 
FROM Product 
WHERE Type = 'jeans' 
AND P_Size = '32' 
AND Gender = 'F' 
AND Quantity > 0;
```

---

###  Modify cart

```sql
DELETE FROM Cart_item 
WHERE Product_id = 'pid1001' 
AND Cart_id IN (
    SELECT Cart_id 
    FROM Customer 
    WHERE Customer_id = 'cid100'
);
```

---

###  Seller stops selling

```sql
DELETE FROM Seller 
WHERE Seller_id = 'sid100';

UPDATE Product 
SET Quantity = 0 
WHERE Seller_id IS NULL;
```

---

###  Products purchased on a particular date

```sql
SELECT Product_id 
FROM Cart_item 
WHERE purchased = 'Y' 
AND Date_Added = '2018-12-12';
```

---

###  Count products sold per date

```sql
SELECT COUNT(Product_id) AS count_pid, Date_Added 
FROM Cart_item 
WHERE purchased = 'Y' 
GROUP BY Date_Added;
```

---

###  Total price in cart

```sql
SELECT SUM(c.Quantity_wished * p.Cost) AS total_payable
FROM Product p
JOIN Cart_item c 
ON p.Product_id = c.Product_id
WHERE c.Cart_id IN (
    SELECT Cart_id 
    FROM Customer 
    WHERE Customer_id = 'cid101'
)
AND c.purchased = 'Y';
```

---

###  Customers who never purchased

```sql
SELECT * 
FROM Customer 
WHERE Customer_id NOT IN (
    SELECT Customer_id FROM Payment
);
```

---

###  Total profit

```sql
SELECT SUM(c.Quantity_wished * p.Cost * p.Commission / 100.0) AS total_profit
FROM Product p
JOIN Cart_item c 
ON p.Product_id = c.Product_id
WHERE c.purchased = 'Y';
```

---

#  Stored Procedures & Functions

### Cost Filter Procedure

```sql
CREATE PROCEDURE cost_filter
    @c DECIMAL(10,2),
    @t VARCHAR(20)
AS
BEGIN
    SELECT Product_id, Cost, Type
    FROM Product
    WHERE Cost < @c AND Type = @t;
END;
```

---

### Total Products Function

```sql
CREATE FUNCTION totalProducts(@sId VARCHAR(6))
RETURNS INT
AS
BEGIN
    DECLARE @total INT;
    SELECT @total = COUNT(*) 
    FROM Product 
    WHERE Seller_id = @sId;
    RETURN @total;
END;
```

---

### Product Details Procedure

```sql
CREATE PROCEDURE prod_details
    @p_id VARCHAR(7)
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Product WHERE Product_id = @p_id)
        SELECT Quantity FROM Product WHERE Product_id = @p_id;
    ELSE
        PRINT 'Sorry no such product exists!';
END;
```

---

#  Triggers

### Customer Insert Trigger

```sql
CREATE TRIGGER before_customer
ON Customer
INSTEAD OF INSERT
AS
BEGIN
    INSERT INTO Cart (Cart_id)
    SELECT Cart_id FROM inserted;

    INSERT INTO Customer
    SELECT * FROM inserted;
END;
```

---

### Payment Auto Total Trigger

```sql
CREATE FUNCTION total_cost(@cId VARCHAR(7))
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @total DECIMAL(10,2);

    SELECT @total = SUM(p.Cost)
    FROM Product p
    JOIN Cart_item c 
    ON p.Product_id = c.Product_id
    WHERE c.Cart_id = @cId;

    RETURN ISNULL(@total, 0);
END;
```

```sql
CREATE TRIGGER before_pay_up
ON Payment
INSTEAD OF INSERT
AS
BEGIN
    INSERT INTO Payment
    SELECT 
        i.payment_id,
        i.payment_date,
        i.payment_type,
        i.customer_id,
        i.cart_id,
        dbo.total_cost(i.cart_id)
    FROM inserted i;
END;
```




