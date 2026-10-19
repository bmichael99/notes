---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/Event/Event.php
---
Back to [[JLab OMS (FOMS)]]

The Events table stores audit logs for all significant actions and state changes within the [[JLab OMS (FOMS)|FOMS]] system. Events are created for any entity ([[FOMS FulfillmentOrder|FulfillmentOrders]], [[FOMS FulfillmentOrderLineItem|LineItems]], etc.) to track who did what and when. This provides a complete audit trail for compliance and debugging purposes.

Events use a universal entity system where `entity_type` and `entity_id` identify what object the event relates to. The `changes` field stores before/after state using JSON for flexible data storage.
- `attributes` diffs are reduced to only the changed keys
- `filters` and `config` diffs are reduced to only the changed keys
- `destinations`, `sources`, and `actions` are reduced by changed index and remain order-sensitive
- `updated_date`-only churn is ignored by shared structure event logging
# Event Types
Events are categorized by their `event_handle`:
- **Order Events**: `submitted`, `confirmed`, `shipped`, `cancelled`
- **Line Item Events**: `line_item_added`, `line_item_updated`, `line_item_removed`
- **System Events**: `created`, `updated`, automated operations, and import/migration side effects
- **Error Events**: `submit_failed`, `destination_confirm_failed`, `destination_confirm_not_found`
This handle corresponds to an [[FOMS EventType|EventType]] record.
# User Attribution
- `user_id = null`: System-generated events
- `user_id = -1`: System user (automated processes)
- `user_id = -2`: Script user (cron jobs, imports)
- `user_id > 0`: Real user ID from authentication system
# Create Table Syntax
```sql
CREATE TABLE IF NOT EXISTS Events (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_type VARCHAR(50) NOT NULL,
    entity_id INT NOT NULL,
    event_handle VARCHAR(100) NOT NULL,
    user_id INT,
    timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    changes JSON,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_entity (entity_type, entity_id),
    INDEX idx_event_handle (event_handle),
    INDEX idx_user_id (user_id),
    INDEX idx_timestamp (timestamp),
    INDEX idx_entity_timestamp (entity_type, entity_id, timestamp)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
# Queries
Get all events for a specific fulfillment order:
```sql
SELECT * FROM Events
WHERE entity_type = 'fulfillment_order'
AND entity_id = ?
ORDER BY timestamp DESC;
```

Find events by user:
```sql
SELECT e.*, et.friendly_name
FROM Events e
LEFT JOIN EventTypes et ON e.event_handle = et.handle
WHERE e.user_id = ?
ORDER BY e.timestamp DESC
LIMIT 50;
```
