attempt to edit a field through a specified path, if it 

```
# nested_payloads = {}

# for key, vals in changes.items():

# # format our payload to meet the specs in https://mydeveloper.logiwa.com/#tag/ShipmentOrder/paths/~1v3.1~1ShipmentOrder~1patch~1%7Bidentifier%7D/patch

# if LOGIWA_SHIPMENT_ORDER_PATCH_MAP[key]["nested"] is None:

# logiwa_post_payload.append({

# "value": vals['to'],

# "path": LOGIWA_SHIPMENT_ORDER_PATCH_MAP[key]["path"],

# "op": "replace"

# })

# else:

# path = LOGIWA_SHIPMENT_ORDER_PATCH_MAP[key]["path"]

# if path not in nested_payloads:

# nested_payloads[path] = {}

# for nested_key, flat_key in LOGIWA_NESTED_OBJECT_SOURCES[path].items():

# nested_payloads[path][nested_key] = order.get(flat_key)

# nested_payloads[path][LOGIWA_SHIPMENT_ORDER_PATCH_MAP[key]["nested"]] = vals["to"]

  

# for path, value in nested_payloads.items():

# logiwa_post_payload.append({

# "value": value,

# "path": path,

# "op": "replace"

# })
```