We are moving away from the TikTok app in Shopify and moving to a direct integration with [[JLab OMS (FOMS)]].

Keys are in [Asana Dev Vault](https://app.asana.com/1/1202559246521129/project/1206201511914098/task/1207818314153419).

https://partner.tiktokshop.com/docv2/page/get-order-list-202309
# FBT
Fulfilled by TikTok (FBT) orders will still have to be imported once they ship.
# FBS
Fulfilled by Seller (FBS) orders will be imported as pending [[FOMS FulfillmentOrder|FulfillmentOrders]] and routed appropriately.
# [[FOMS InventoryFeed|InventoryFeed]]
We need to create a FOMS InventoryFeed to push the new inventory to tiktok directly:
https://partner.tiktokshop.com/docv2/page/update-inventory-202309
- requires the SKU ID which can be got using the sku from a separate call
```bash
- https://partner.tiktokshop.com/docv2/page/search-products-202502curl -X POST \
 'https://open-api.tiktokglobalshop.com/product/202309/products/1729382561353664372/inventory/update?timestamp=1623812664&shop_cipher=GCP_XF90igAAAABh00qsWgtvOiGFNqyubMt3&app_key=38abcd&sign=5361235029d141222525e303d742f9e38aea052d10896d3197ab9d6233730b8c' \
-H 'x-tts-access-token: TTP_pwSm2AAAAABmmtFz1xlyKMnwg74T2GJ5s0uQbS8jPjb_GkdFVCxPqzQXSyuyfXdQa0AqyDsea2tYFNVf4XeqgZHFfPyv0Vs659QqyLYfsGzanZ5XZAin3_ZkcIxxS0_In6u6XDeU96k' \
-H 'content-type: application/json' \
-d '{
  "skus": [
    {
      "id": "1729592969712207013",
      "inventory": [
        {
          "warehouse_id": "7068517275539719942",
          "quantity": 999,
          "backorder_quantity": 888,
          "handling_time": 5
        }
      ]
    }
  ]
}'
```
- requires the SKU ID which can be got using the sku from a separate call
- https://partner.tiktokshop.com/docv2/page/search-products-202502
# Fulfillment
Split order
```bash
curl -X POST \
 'https://open-api.tiktokglobalshop.com/fulfillment/202309/orders/556643423444/split?app_key=38abcd&sign=5361235029d141222525e303d742f9e38aea052d10896d3197ab9d6233730b8c&timestamp=1623812664&shop_cipher=GCP_XF90igAAAABh00qsWgtvOiGFNqyubMt3' \
-H 'x-tts-access-token: TTP_pwSm2AAAAABmmtFz1xlyKMnwg74T2GJ5s0uQbS8jPjb_GkdFVCxPqzQXSyuyfXdQa0AqyDsea2tYFNVf4XeqgZHFfPyv0Vs659QqyLYfsGzanZ5XZAin3_ZkcIxxS0_In6u6XDeU96k' \
-H 'content-type: application/json' \
-d '{
  "splittable_groups": [
    {
      "id": "123",
      "order_line_item_ids": [
        "57646237751283022",
        "576462377512830223"
      ]
    }
  ]
}'
```
Fulfill order
```bash
curl -X POST \
 'https://open-api.tiktokglobalshop.com/fulfillment/202309/packages/ship?app_key=38abcd&sign=5361235029d141222525e303d742f9e38aea052d10896d3197ab9d6233730b8c&shop_cipher=GCP_XF90igAAAABh00qsWgtvOiGFNqyubMt3&timestamp=1623812664' \
-H 'x-tts-access-token: TTP_pwSm2AAAAABmmtFz1xlyKMnwg74T2GJ5s0uQbS8jPjb_GkdFVCxPqzQXSyuyfXdQa0AqyDsea2tYFNVf4XeqgZHFfPyv0Vs659QqyLYfsGzanZ5XZAin3_ZkcIxxS0_In6u6XDeU96k' \
-H 'content-type: application/json' \
-d '{
  "packages": [
    {
      "id": "12321312312431",
      "handover_method": "PICKUP",
      "pickup_slot": {
        "start_time": 1623812664,
        "end_time": 1623812664
      },
      "self_shipment": {
        "tracking_number": "JX12345",
        "shipping_provider_id": "6617675021119438849"
      }
    }
  ]
}'
```