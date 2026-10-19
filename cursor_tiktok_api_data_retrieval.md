# TikTok API data retrieval
_Exported on 9/28/2026 at 15:55:04 PDT from Cursor (3.20.21)_

---

**User**

@python_code/tiktok/invoicing_us_v2/docs/income_20260925095305(UTC-7).xlsx It looks like the Tik Tok API does not provide all the data that's provided by the xlsx, is there a different API endpoint we can hit to retrieve all of this matching data for orders?

---

**Cursor**

The income spreadsheet and the current TikTok order pull look like they cover different fields. I’ll compare the spreadsheet columns with what the code requests today, then check whether another TikTok endpoint returns the matching order data.

The spreadsheet is a settlement income export, not an order export. I’ll check the current TikTok Finance endpoints against those columns so we can see which call actually returns the matching order-level amounts.

Yes. That spreadsheet is the Seller Center settlement report, and the matching data is on the Finance API, not on another Orders endpoint.

`POST /order/202309/orders/search` is what the current client calls. That payload is the order at checkout: status, buyer, address, SKU, and a small `payment` object (list price, discounts, shipping, tax, total). The income file is a later ledger. Its Order details sheet is one row per settled SKU, plus adjustment rows, dated by statement date. In this file that is 1,259 Order rows, 2 FBT warehouse-fee rows, and 1 logistics reimbursement, all with status Paid. An order created in June can show up on an August statement, so a create-time order pull will never reproduce this file.

## Endpoint to use

Pull statements for the export window, then the transactions on each statement. That is the same grain as the Order details sheet, and it includes adjustments that are not orders.

| What you need | Endpoint | Scope |
|---|---|---|
| Statement list (Statements sheet: statement id, date, status, payment id, net sales, shipping, fees, adjustments) | `GET /finance/202309/statements` | `seller.finance.info` |
| SKU settlement lines (Order details sheet) | `GET /finance/202501/statements/{statement_id}/statement_transactions` | `seller.finance.info` |
| Same breakdown when you already have an order id | `GET /finance/202501/orders/{order_id}/statement_transactions` | `seller.finance.info` |
| Money not on a statement yet | `GET /finance/202507/orders/unsettled` | `seller.finance.info` |
| Payouts (Payments sheet) | `GET /finance/202309/payments` | `seller.finance.info` |

Use the 202501 transaction endpoints. The older 202309 transaction paths exist, but they do not return the SKU-level fee breakdown this export is built from.

Filter statements by `statement_time`, not order create time. Docs: [Get Statements](https://partner.tiktokshop.com/docv2/page/get-statements-202309), [Get Transactions by Statement (202501)](https://partner.tiktokshop.com/docv2/page/get-transactions-by-statement-202501), [Get Transactions by Order (202501)](https://partner.tiktokshop.com/docv2/page/get-transactions-by-order-202501). The app also has to be re-authorized with `seller.finance.info`. Order scopes do not cover these calls.

Call the order endpoint only when you are looking up orders you already have. One order can land on more than one statement (the original settlement, then a refund). Calling it once per order also misses adjustment rows whose id is an adjustment id, with the order only in Related order ID.

## How the columns map

On each statement transaction, `sku_transactions[]` is the Order row:

- Statement ID, currency, type, order id, SKU id, quantity, product name, SKU name, order created time
- Total settlement amount → `settlement_amount`
- Net sales → `revenue_amount`
- Gross sales, gross sales refund, seller discount, seller discount refund → `revenue_breakdown`
- Shipping and its components (customer-paid shipping, FBT fulfillment fee, signature fee, insurance, return shipping, subsidies) → `shipping_cost_amount` and `shipping_cost_breakdown`, including `supplementary_component`
- Transaction fee, referral fee, refund administration fee, affiliate commission, partner commission, shop ads commission, co-funded promotion, GMV Max, Smart Promotion → `fee_tax_breakdown.fee` and `.tax`
- Customer payment, customer refund, platform discounts, co-funded voucher discounts, sales tax, retail delivery fee → `supplementary_component`
- Adjustment amount, adjustment id, and related order id are on the transaction itself when `type` is not `ORDER`

Statement date, payment status, and Payment ID come from the statements list (`statement_time`, `payment_status`, `payment_id`), joined on statement id. The Payments and Reserve sheets are the payments endpoint and the reserve fields on those same transactions, not the Orders API.

A few columns on the far right of Order details are still only on the spreadsheet or on the order/fulfillment APIs: order shipment date, order delivery date, package weight, collection method, delivery option, and the shipping-discount mode (the sample value is `Co-funded`). Newer campaign-fee labels (Growth Package fees, Brand Amplification, auto-post video commission) show up as additional fee fields when TikTok adds them; they are not on `orders/search`.

---

**User**

So what I should probably do is do a RAW->BRONZE->SILVER with the GET /finance/202309/statements that we run every day. That way we can accurantely create invoices and credit memos for an order number, and apply those to the correct invoice based on the NETSUITE_DOCUMENTS table, which will allow us to connect an order number to its netsuite documents

---

**Cursor**

Statements are the right daily driver, but the statement list alone does not carry the order-level amounts you need for credit memos. I’ll check how bronze, silver, and `NETSUITE_DOCUMENTS` are set up so the pipeline matches that join.

The daily RAW → BRONZE → SILVER job is the right shape, and `GET /finance/202309/statements` is the right thing to poll. The order amounts you need for credit memos are one call further down.

A statement is the settlement batch: statement id, statement date, payment status, payment id, and the batch totals. It has no order number, SKU, fee, refund, or adjustment reason. After each statement comes back, pull `GET /finance/202501/statements/{statement_id}/statement_transactions` and store that payload beside it. That is the Order details grain from the income file: one row per settled SKU, plus adjustment rows, each carrying the TikTok order id.

Store both the same way orders are stored now.

- **RAW.** One ingest row per run: statement-time window, counts, file hash, status.
- **BRONZE.** One row per distinct statement payload, and one row per distinct transaction payload, keyed by a hash so a status change (`Pending` → `Paid`) or a later adjustment is a new version.
- **SILVER.** `STATEMENT` (one row per statement id), `STATEMENT_TRANSACTION` (order or adjustment, with `order_id` and related order id), and `STATEMENT_TRANSACTION_SKU` (the fee, discount, refund, and shipping amounts).

Poll a rolling window, not only yesterday. A refund or fee for an order created in June can land on an August statement, and the same statement id changes status after the first pull. Re-read recent statements and let the hash keep the new version.

Invoices stay on the path you already have. The morning order pull builds the sales order, FOMS shipments build the item fulfillment, and closing that fulfillment creates the invoice and writes `NETSUITE_DOCUMENT_LINES`. Settlement happens after delivery and the return window, so a statement is often days or weeks later than the invoice. Waiting on it to create the invoice would miss that close.

The finance silver table is what you post after the invoice exists:

1. Group each order’s statement lines into the credit-memo categories you already defined: seller discount, fees, and returns. Leave out platform discounts TikTok funds, and leave out shipping that nets to zero.
2. Join `order_id` to `NETSUITE_DOCUMENT_LINES.tiktok_order_id` where `document_type` is `INVOICE`. The SKU on the statement line picks the invoice line when one order shipped across more than one invoice.
3. Apply the credit memo and the payment to that invoice’s `netsuite_internal_id`.
4. Keep the order-level rows in Snowflake. NetSuite only stores the category totals, which is the audit trail accounts receivable needs.

When the original invoice’s period is already closed, the statement still tells you the order and the amount. The posting rule then puts that credit memo on an open invoice and records the substitution. That exception is a decision in the posting job, and the statement row remains the evidence of which order it belonged to.

---

**User**

should I still use the other orders api that's currently used in the tiktok api or just transition over to using these 2 endpoints

---

**Cursor**

Keep the Orders API. Add the two Finance endpoints beside it. They cover different days in the same order’s life, and neither one has the fields the other job needs.

The morning job still has to come from `POST /order/202309/orders/search`. That is the only call that knows which orders were created yesterday, while they are still awaiting shipment. Silver `ORDER` and `ORDER_LINES` are built from it: `create_time` picks the day’s sales order, `seller_sku` and `warehouse_id` become the NetSuite line and location, `original_price` is the rate, and `status` is what cancels a line or closes the sales order. Item fulfillments and invoices are created from those lines as FOMS reports shipments. A statement does not exist yet at that point. Settlement waits until delivery plus the return window, so yesterday’s orders will not be on `GET /finance/202309/statements` for days or weeks.

The finance pair is the later job. Once a statement is paid, its transactions carry the order id plus the seller discount, fees, refunds, and settlement amount you turn into credit memos and the payment. Those post onto the invoice already stored in `NETSUITE_DOCUMENT_LINES`. The statement list has none of the line fields the sales order needs, and the order payload has none of the fee breakdown the credit memo needs.

| Job | Endpoint | When it runs |
|---|---|---|
| Sales order, cancellation, item fulfillment, invoice | `POST /order/202309/orders/search` | Morning, for orders created yesterday |
| Credit memos and payment on that invoice | `GET /finance/202309/statements`, then `GET /finance/202501/statements/{statement_id}/statement_transactions` | Daily, over a rolling statement-time window |

---

**User**

do we have any code interacting with the new endpoints we found for building invoices?

---

**Cursor**

I’ll search the TikTok invoicing code for any calls to the Finance statement endpoints.

No. Nothing in this repo calls the Finance endpoints.

The only TikTok API client is `python_code/tiktok/invoicing_us_v2/src/etl/tiktok_api.py`, and the only path it requests is `POST /order/202309/orders/search` through `fetch_orders` and `fetch_orders_by_date_range`. There are no references to `/finance/202309/statements`, `/finance/202501/statements/{statement_id}/statement_transactions`, settlement amounts, or `seller.finance.info`.

Invoice creation is not wired to those endpoints either. The v2 NetSuite scripts only stub the `NETSUITE_DOCUMENTS` tables, and the silver transform still builds `ORDER` and `ORDER_LINES` from the order payload.

---

**User**

so nothing in invoicing_us (v1)

---

**Cursor**

I’ll check `invoicing_us` on its own so the v1 answer isn’t just inferred from the v2 search.

Correct. `invoicing_us` never calls the Finance endpoints either.

Its client in `python_code/tiktok/invoicing_us/src/etl/tiktok_api.py` only requests `POST /order/202309/orders/search`. The tests and signature scripts use that same path. There are no references to statements, statement transactions, or settlement amounts anywhere in v1.

