---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/Router/FomsRouter.php
---
This is the FOMS Routers table. It holds routing configuration for order processing workflows.
# Create Table Syntax
```sql
CREATE TABLE `Routers` (
  `id` int NOT NULL AUTO_INCREMENT,
  `handle` varchar(255) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `description` text,
  `enabled` tinyint(1) DEFAULT '1',
  PRIMARY KEY (`id`),
  UNIQUE KEY `handle` (`handle`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
