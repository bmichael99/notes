---
project: FOMS
type: model
php_file: vendor/joeybab3/router/src/Criteria.php
---
This is the FOMS Criteria table. It holds conditions for business [[FOMS Rule|Rules]] in the [[FOMS Router|Routing]] system.
# Create Table Syntax
```sql
CREATE TABLE `Criteria` (
  `id` int NOT NULL AUTO_INCREMENT,
  `rule_id` int NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `description` text,
  `conditions` json,
  `priority` int DEFAULT NULL,
  `enabled` tinyint(1) DEFAULT '1',
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `rule_id` (`rule_id`),
  CONSTRAINT `Criteria_ibfk_1` FOREIGN KEY (`rule_id`) REFERENCES `Rules` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
