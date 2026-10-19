---
project: FOMS
type: model
php_file: vendor/joeybab3/router/src/Rule.php
---
This is the FOMS Rules table. It holds business rules for order routing and processing.
# Create Table Syntax
```sql
CREATE TABLE IF NOT EXISTS Rules (
    id INT AUTO_INCREMENT PRIMARY KEY,
    router_id INT NOT NULL,
    name VARCHAR(255),
    description TEXT,
    actions JSON,
    priority INT,
    enabled BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (router_id) REFERENCES Routers(id)
)
```
