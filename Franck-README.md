

Here are my tests

## initialization

MongoDB:
```
mongosh -f data/init_mongo.js
```

Data:
```
python3 -m venv path/to/venv  
source path/to/venv/bin/activate  
pip install -r requirements.txt 

python3 data/data_loader.py    
```

## New use case: Top-10 last orders in state with product

```
explain (analyze, buffers)
SELECT   
    o.id AS order_id,  
    o.order_date AS order_date,  
    c.first_name AS customer_first_name,  
    c.last_name AS customer_last_name,  
    p.name AS product_name,  
    od.quantity AS quantity,  
    a.state AS address_state  
FROM   
    orders AS o  
INNER JOIN   
    order_details AS od ON o.id = od.order_id  
INNER JOIN   
    products AS p ON od.product_id = p.id  
INNER JOIN   
    customers AS c ON o.customer_id = c.id  
INNER JOIN   
    addresses AS a ON o.address_id = a.id  
WHERE   
    p.name LIKE 'Vodka%'   
    AND a.state = 'CA'  
ORDER BY   
    o.order_date DESC  
LIMIT   
    10;  

 order_id |         order_date         | customer_first_name | customer_last_name | product_name  | quantity | address_state 
----------+----------------------------+---------------------+--------------------+---------------+----------+---------------
 13587151 | 2024-12-28 18:24:29.821515 | Cheryll             | Knight             | Vodka Sunrise |        7 | CA

```

Possible indexes (but none will be perfect):
```
CREATE INDEX idx_products_name ON products (name);  
CREATE INDEX idx_addresses_state_id ON addresses (state, id);  
CREATE INDEX idx_order_details_order_product ON order_details (order_id, product_id);  
CREATE INDEX idx_orders_date ON orders (order_date DESC, id);  
CREATE INDEX idx_orders_address_date ON orders (address_id, order_date DESC);  
CREATE INDEX idx_order_details_covering ON order_details (order_id, product_id, quantity);  
```

The same in MongoDB

```
db.orders.find(  
    {  
        "shippingAddress.state": "CA",  
        "details.product.name": "Vodka Sunrise"   
    },  
    {  
        _id: 1,  
        orderDate: 1,  
        "customer.firstName": 1,  
        "customer.lastName": 1,  
        "details.product.name": 1,  
        "details.quantity": 1,  
        "shippingAddress.state": 1,  
        total: 1  
    }  
).sort({ orderDate: -1 }).limit(1).explain("executionStats").executionStats;  
```

This simple index finds the result immediately:
```
db.orders.createIndex({ "shippingAddress.state": 1, "details.product.name": 1, "orderDate": -1 });  
```
