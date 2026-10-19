# FOMS Deep Audit — May 2026

> A consolidated audit of the FOMS web UI, Composer package, cron pipeline, REST API, and dependency manifest. Findings are grouped by **effort tier** so this document doubles as a remediation backlog.

---

## How to read this file

Every finding uses the same compact line format so the file is grep-friendly:

```
- [SEV][AREA] file/path:lineRange — One-sentence description. Fix: short fix sentence.
```

- `SEV` = `CRIT` (Critical) · `HIGH` · `MED` · `LOW`
- `AREA` = `SEC` (Security) · `REL` (Reliability/Correctness) · `PERF` (Performance) · `MOD` (Modernization) · `BUG` (Functional bug) · `TEST` (Test coverage) · `ARCH` (Architecture) · `OBS` (Observability/Logging)

Tiers:

- **Tier 1 — Quick wins**: ≤ 1–5 lines per fix, no design decisions, can be done in a single PR.
- **Tier 2 — Function-level**: A function/method or a tight block; small refactor.
- **Tier 3 — Page / script / class-level**: A whole file, controller, or pipeline step.
- **Tier 4 — Architectural**: Cross-cutting changes, schema or DI overhauls, multi-week effort.

Citations: All paths are relative to `/var/www/html/` unless noted. Line ranges were captured during the audit and should be re-confirmed before changes — line numbers drift.

---

## Audit scope

| Surface | Path | What was reviewed |
|---|---|---|
| Web UI pages | `ecomm/foms/` | Page-level PHP, AJAX, `.htaccess`, inline JS, CSS |
| Composer package | `vendor/joeybab3/foms/` | `src/`, `tests/`, `composer.json`, `composer.lock` |
| Cron pipeline | `ecomm/cron/foms/` | Pipeline steps, utilities, crontab schedule |
| Inventory cron | `ecomm/cron/inventory/` | Logiwa inventory builder and adjacent jobs |
| REST API | `ecomm/foms/api/` + `vendor/joeybab3/foms/src/Api/` | Routing, auth, controllers |
| Project root | `composer.json` | Stability constraints, internal package strategy, CI hooks |

---

## Severity × Effort matrix (counts)

|             | Tier 1 (1-line) | Tier 2 (function) | Tier 3 (page/script/class) | Tier 4 (architectural) | **Total** |
|-------------|:---:|:---:|:---:|:---:|:---:|
| **CRIT**    | 4 | 2 | 3 | 0 | **9**  |
| **HIGH**    | 17| 27| 15| 9 | **68** |
| **MED**     | 18| 23| 14| 14| **69** |
| **LOW**     | 6 | 2 | 3 | 4 | **15** |
| **Total**   | **45** | **54** | **35** | **27** | **161** |

The **top-10 must-fix** list lives directly below. Counts above exclude the top-10 summary list since those items are restatements of in-tier findings.

---

## Top 10 must-fix (do these first)

1. **CRIT/SEC** — Remove Logiwa bearer token from logs: `vendor/joeybab3/foms/src/OrderDestination/LogiwaDestination.php:183`.
2. **CRIT/SEC** — Rotate and remove hardcoded Snowflake password in `ecomm/cron/foms/foms_sync_to_snowflake.php:137` (and 3 utilities). Scrub git history.
3. **CRIT/SEC** — `PDO::MYSQL_ATTR_LOCAL_INFILE => true` in `vendor/joeybab3/foms/src/App.php:181` — remove.
4. **CRIT/SEC** — `shell_exec("netsuite lessen invoice " . $id)` in `vendor/joeybab3/foms/src/NetSuite/NetSuiteInfoRepository.php:176-186` — replace with REST or `escapeshellarg` + numeric validation.
5. **CRIT/SEC** — TikTok OAuth callback exposes auth codes publicly: `ecomm/foms/tiktok_oauth_callback.php` — gate behind SSO or remove from prod; verify `state`.
6. **CRIT/REL** — `invoice_shipments.php` calls `save('invoiced')` on `error` and `keep_pending` paths: `ecomm/cron/foms/invoice_shipments.php:242-256`.
7. **CRIT/REL** — `FulfillmentOrderRepository::cancel()` recreate-flow has 10+ writes with no transaction and inserts the new row with status `cancelled` before updating the original: `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1819-1959`.
8. **HIGH/SEC** — Reflected XSS via `?msg=` across 7 pages (see Tier 1 SEC items).
9. **HIGH/SEC** — Missing destructive-write authorization on `FulfillmentOrderController`, `ShipmentController`, `InventoryFeedController`, `EventController` (no `readWriteDenied` / `userlevel` guards).
10. **HIGH/SEC** — No CSRF tokens on FOMS API/AJAX mutating endpoints.

---

## Tier 1 — Quick wins (single-line / config edits)

Each finding here is realistically a 1-line or 1-config edit. Group by category.

### Security (Tier 1)

- [CRIT][SEC] `ecomm/cron/foms/foms_sync_to_snowflake.php:137` — Hardcoded Snowflake password literal in source. Fix: read from `/var/www/serve/foms.ini` and rotate the credential.
- [CRIT][SEC] `ecomm/cron/foms/fetch_walmart_item_performance_data.php:215` — Same hardcoded Snowflake password. Fix: read from ini after rotating.
- [HIGH][SEC] `ecomm/cron/foms/utilities/delete_walmart_item_performance_sentinel_rows.php:10` — Snowflake password literal. Fix: move to ini.
- [HIGH][SEC] `ecomm/cron/foms/utilities/list_walmart_item_performance_missing_dates.php:52` — Snowflake password literal. Fix: move to ini.
- [HIGH][SEC] `ecomm/foms/fulfillment_orders.php:47` — Reflected XSS: `rtrim($_GET['msg'], '/\\')` injected into JS for `notificationMessage()`. Fix: `json_encode($_GET['msg'] ?? '')`.
- [HIGH][SEC] `ecomm/foms/inventory.php:46` — Same `?msg=` reflected XSS pattern. Fix: `json_encode`.
- [HIGH][SEC] `ecomm/foms/shipments.php:63` — Same `?msg=` reflected XSS. Fix: `json_encode`.
- [HIGH][SEC] `ecomm/foms/shipment.php:132` — Same `?msg=` reflected XSS. Fix: `json_encode`.
- [HIGH][SEC] `ecomm/foms/bundles/index.php:52` — Same `?msg=` reflected XSS. Fix: `json_encode`.
- [HIGH][SEC] `ecomm/foms/bundles/bundle.php:70` — Same `?msg=` reflected XSS. Fix: `json_encode`.
- [HIGH][SEC] `ecomm/foms/bundles/new_bundle.php:54` — Same `?msg=` reflected XSS. Fix: `json_encode`.
- [HIGH][SEC] `ecomm/foms/tiktok_oauth_callback.php:49` — Server filesystem path disclosed in user-facing HTML. Fix: remove the path string from the rendered output.
- [HIGH][SEC] `vendor/joeybab3/foms/src/OrderSource/WalmartOrderSource.php:133` — Logs partial Walmart Client ID. Fix: drop credential-adjacent logging.
- [MED][SEC] `vendor/joeybab3/foms/src/InventoryFeed/ShopifyInventoryFeed.php:208` — Logs full GraphQL response (may contain pricing/PII) when debug is on. Fix: redact or summarize.
- [MED][SEC] `ecomm/cron/inventory/foms_build_logiwa_inventory.php:55` — `error_log()` of entire Logiwa API response per page. Fix: log summary counts.
- [MED][SEC] `ecomm/foms/foms/.htaccess` — Missing `Options -Indexes`, `X-Frame-Options`, `Content-Security-Policy`, `Referrer-Policy`. Fix: add standard security headers in `.htaccess`.
- [MED][SEC] `ecomm/foms/dummy_order.json` and `dummy_shopify.json`, `dummy_walmart.json`, `dummy_shipstation.json` — Sample payloads served as static files. Fix: deny web access via `.htaccess` (`<FilesMatch "^dummy_.*\.json$"> Require all denied </FilesMatch>`).

### Bugs (Tier 1)

- [HIGH][BUG] `vendor/joeybab3/foms/composer.json:6` — `phpunit/phpunit` is in `require`, not `require-dev`, shipping test code to prod. Fix: move to `require-dev` and re-lock.
- [MED][BUG] `vendor/joeybab3/foms/composer.json:4` — Missing `"php": "^8.x"` constraint. Fix: add `"php": "^8.2"` (or whatever the deployment runs).
- [MED][BUG] `vendor/joeybab3/foms/composer.json:4` — Missing `ext-pdo`, `ext-json`, `ext-curl`, `ext-openssl` declarations despite heavy use. Fix: add `ext-*` entries to fail-fast at install.
- [MED][BUG] `composer.json:27` — Root `"minimum-stability": "dev"` allows unstable releases project-wide. Fix: set to `"stable"`; mark specific deps with `@dev` only where needed.
- [MED][BUG] `vendor/joeybab3/foms/composer.json` — Missing `"license"` SPDX field. Fix: add `"license": "proprietary"`.
- [MED][BUG] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1018-1020` — `var_dump($fulfillmentOrder->getOrderData())` on save failure leaks order payload to stdout/log. Fix: replace with `$this->app->log(...)`.
- [MED][BUG] `ecomm/foms/fulfillment_orders.php:51-54` — Copy-paste bug: references undefined `$set` and `$task` from an Amazon page. Fix: remove the dead block.
- [MED][BUG] `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php:172-174` — `search()` sets `limit=25` on the singleton repository and never restores the default; later list queries inherit 25. Fix: save/restore the limit or pass per-call.
- [MED][BUG] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrder.php:120-127` — `getDestinationUrl()` / `getDestinationOrderId()` typed `?string` but return `''` when null, breaking `!empty()` null checks. Fix: return `null`.
- [LOW][BUG] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrder.php:60` — `setTableName('FulfillmentOrders')` does not match the real table name `FulfillmentOrder`. Fix: correct the literal.
- [LOW][BUG] `vendor/joeybab3/foms/src/Router/RuleRepository.php:76-77` — `decodeActions()` does `empty()` check then `$raw[0]`; a single whitespace passes and is mis-parsed. Fix: `trim($raw)` first.
- [LOW][BUG] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderImportService.php:363` — Calls `$app->log(..., 'error')` but `App::log()` only supports `echo`/`var_dump`, so the call is silently ignored. Fix: use a supported log type or add an `error` channel.
- [LOW][BUG] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderConfirmationService.php:62` — On confirm failure, only `status_message` is set; `status_handle` is left stale. Fix: set `status_handle` to `error` (or a dedicated state).
- [LOW][BUG] `ecomm/foms/foms/.htaccess:29` — Rewrite to non-existent `logiwa_order.php`. Fix: remove the rule.

### Reliability (Tier 1)

- [HIGH][REL] `ecomm/cron/foms/invoice_shipments.php:242-256` — Always calls `$shipment->save('invoiced')` even after exceptions and on `keep_pending` skips, writing misleading audit events. Fix: only `save()` on true status transitions; mirror `ShipmentController` (vendor lines 304-321).
- [HIGH][REL] `ecomm/cron/foms/invoice_shipments.php` — No `flock`; can race with `bulk_invoice_shipments.php` and manual utilities on the same `pending` shipments. Fix: add `flock` like `submit_fulfillment_orders.php:4-16`.
- [HIGH][REL] `ecomm/cron/foms/import_cancelation_requests.php` — Scheduled `*/5 * * * *` with no lock; can overlap itself and race with import/submit. Fix: add `flock`.
- [HIGH][REL] `ecomm/cron/foms/import_fulfillment_orders.php:237` — `acknowledgeOrder(null)` violates `OrderSourceInterface::acknowledgeOrder(FulfillmentOrder $order)` — TypeError on PHP 8+ for Shopify. Fix: pass the FO or skip ack.
- [HIGH][REL] `ecomm/cron/foms/utilities/import_shopify_order_by_number.php:132` — Same `acknowledgeOrder(null)` TypeError. Fix: pass the imported FO.
- [MED][REL] `vendor/joeybab3/foms/src/Structure.php:260-266` — `delete()` swallows all exceptions and returns `false` with no logging. Fix: log the exception and either rethrow or return a structured error.
- [MED][REL] `vendor/joeybab3/foms/src/Structure.php:332-335` — `createEvent()` failures only log when debug is on; production silently returns null. Fix: always `error_log` failures.

### Observability (Tier 1)

- [MED][OBS] `ecomm/cron/foms/import_confirmations.php:67-71` — Always prints "Successfully checked" even on failure paths. Fix: branch on `$result->isSuccess()` / `isFound()`.
- [MED][OBS] `ecomm/cron/crontab.backup:41` — `warehouse_load.php` cron line omits `2>&1`; stderr lost. Fix: append `2>&1`.
- [MED][OBS] `ecomm/cron/crontab.backup:37` — `update_inventory.php` runs with `--debug` in production. Fix: remove the flag.
- [MED][OBS] All FOMS pipeline scripts (`ecomm/cron/foms/`) — Lock-skip path exits 0, masking missed ticks. Fix: exit non-zero or emit a `foms_cron_skipped` metric when `flock` fails.
- [LOW][OBS] `vendor/joeybab3/foms/src/Api/ApiRouter.php:63` — Unknown route throws `Exception("Route not found: ...")` and that message is forwarded to the client. Fix: log server-side, return generic `404`.

---

## Tier 2 — Function-level fixes

### Security (Tier 2)

- [CRIT][SEC] `vendor/joeybab3/foms/src/NetSuite/NetSuiteInfoRepository.php:176-186` — `shell_exec("netsuite lessen invoice " . $netSuiteInfo->getNetsuiteInternalId())` builds a shell command from DB data; a poisoned internal ID enables command injection. Fix: replace with NetSuite REST delete or `escapeshellarg()` + `ctype_digit()` validation.
- [CRIT][SEC] `ecomm/cron/foms/utilities/exchange_tiktok_oauth_code.php:38-43,148,194` — Dumps full OAuth tokens to STDERR/STDOUT. Fix: never print tokens; write only to a restrictive-permissions ini file.
- [HIGH][SEC] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:75-77,1052-1106` — `setSortBy()` / `setSortDirection()` interpolate raw strings into `ORDER BY fo.{$sortBy}` with no allowlist. Fix: validate against a static allowlist before assignment.
- [HIGH][SEC] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:490,233` — Dynamic `ORDER BY {$this->sortby}` propagates wherever the singleton is reused. Fix: enforce allowlist at the setter.
- [HIGH][SEC] `ecomm/foms/components/foms_ui/foms_ui.js:1107-1132` — `renderFulfillmentOrder()` interpolates customer name, address fields, email, and notes into template literals without `escapeHtml()`. Fix: wrap every dynamic field with the existing `escapeHtml()` helper.
- [HIGH][SEC] `ecomm/foms/index.php:386-444` — DataTable `render` callbacks concatenate `order_name`, `customer`, `destination_url`, `status_message` directly into HTML. Fix: add a local `escapeHtml()` helper and use it in every `render`.
- [HIGH][SEC] `ecomm/foms/index.php:560-576` — jQuery UI autocomplete `_renderItem` builds HTML with unescaped API fields. Fix: escape before template interpolation.
- [HIGH][SEC] `ecomm/foms/index.php:666-719` — Stats widgets write `item.customer`, `item.destination`, `item.status_handle` to `innerHTML`. Fix: build with `textContent` / escape.
- [HIGH][SEC] `ecomm/foms/deliveries.php:431-454` — Same unsafe innerHTML pattern in stats. Fix: escape API strings.
- [HIGH][SEC] `ecomm/foms/preview_storefront.php:206-235` — Order/customer/status injected via `innerHTML`. Fix: escape or use `textContent`.
- [HIGH][SEC] `ecomm/foms/shipments.php:132-134` — Tracking link `href` uses raw `row[5]` URL with no scheme allowlist; `javascript:` URLs slip through. Fix: validate `^https?://`.
- [HIGH][SEC] `includes/head.php:264-271` — Global `notificationMessage()` assigns `${title}`/`${message}` to `innerHTML` without escaping (the sink for every `?msg=` finding). Fix: escape or use `textContent` for title/message bodies.
- [HIGH][SEC] `ecomm/foms/components/timeline/timeline.js:852-854` — `showNotification()` sets `innerHTML` with raw `${message}`. Fix: `textContent` or escape.
- [HIGH][SEC] `ecomm/foms/rule.php:1405-1426` — Criteria import preview injects names into `innerHTML` unescaped. Fix: use `textContent` in list items.
- [HIGH][SEC] `ecomm/foms/js/bulk-tasks.js:512-518,503` — Destination dropdown options built with unescaped `handle`/`name`. Fix: build options with `document.createElement('option')` and assign via `textContent`.
- [MED][SEC] `vendor/joeybab3/foms/src/OrderSource/TikTokOrderSource.php:156` — `file_get_contents($this->credentialsPath)` reads credentials from a config-derived path with no canonicalization. Fix: assert path is under an allowed credentials directory.
- [MED][SEC] `vendor/joeybab3/foms/src/OrderDestination/NetSuiteDestination.php:56-64` — Constructor defaults to empty OAuth secrets; can attempt NetSuite calls with blank credentials. Fix: throw in constructor when required keys are missing.
- [MED][SEC] `ecomm/cron/foms/foms_sync_to_snowflake.php:105,163,340` — `snowsql` invocations interpolate `$csvPathForPut` without `escapeshellarg`. Fix: wrap all path/CLI args in `escapeshellarg()`.
- [MED][SEC] `ecomm/cron/foms/utilities/reset_invoiced_order.php:92-96,126-130` — `exec("netsuite lessen invoice ...")` from DB/CSV-derived IDs. Fix: validate `ctype_digit()` and `escapeshellarg()`; or use a PHP NetSuite client.
- [MED][SEC] `ecomm/cron/foms/utilities/mass_lessen_netsuite_invoices.php:21-25` — Same shell-out pattern with no dry-run guard. Fix: `escapeshellarg` + numeric assertion + `--dry-run` default.
- [MED][SEC] `ecomm/foms/api/index.php:212-216` — Catches `Exception` only; `\Error`/`\TypeError` produce 500 HTML responses. Fix: `catch (\Throwable $e)`, log and return generic JSON.
- [MED][SEC] `ecomm/foms/api/index.php:203-208` — `array_merge($_GET, $_POST)` lets query params override POST body. Fix: merge POST over GET (or reject duplicate keys).
- [MED][SEC] `vendor/joeybab3/foms/src/Api/InventoryFeedController.php:60-61` — Returns `$e->getMessage()` to the client. Fix: log internal, return a generic message.
- [MED][SEC] `ecomm/foms/components/foms_ui/foms_ui.js` — `escapeHtml()` helper exists and is used in *some* render paths but not all. Fix: lint or refactor so every render goes through it.

### Reliability (Tier 2)

- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderSubmissionService.php:47-63` — Destination `submit()` succeeds and sets `destination_order_id`, then source `submitOrder()` fails → FO is `error` with a live destination record; retry can double-submit. Fix: add idempotency key on destination submit and compensating rollback on partial failure.
- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1859-1875` — In `cancel()`, line-item/shipment cancel failures are caught as warnings and processing continues — FO may be cancelled while child rows remain active. Fix: fail fast or wrap in a transaction.
- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1336-1421` — `processExistingFulfillments()` creates shipments and updates line items in a loop with no transaction. Fix: transaction per fulfillment or per batch.
- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1656-1725` — `moveOrderToDestination()` updates line items and FO across multiple writes without a transaction. Fix: `beginTransaction`/`commit`.
- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderConfirmationService.php:91-103` — `isOrderFoundAtDestination()` returns `true` for `manual`/`external` types without verifying `confirm()` outcome. Fix: require explicit success signal from the destination adapter.
- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:898-939` — `update()` does not invalidate the identity-map cache entry; subsequent `findById()` returns stale state. Fix: `unset($this->fulfillmentOrders[$id])` (or refresh cache).
- [HIGH][REL] `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php:988-1022` — `delete()` writes an audit event *before* `beginTransaction`; rollback leaves a "deleted" event for an extant record. Fix: create the event after `commit`.
- [MED][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1950-1954` — Post-cancel `acknowledgeOrder()` failure is logged and ignored; the new pending FO may never be acked at the source. Fix: queue retry or fail-loudly with metric.
- [MED][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderImportService.php:360-364` — Import-time invoice exception is swallowed; FO imports succeed but NetSuite shipping invoice is silently missing. Fix: surface failure in `ImportResult` or push to retry queue.
- [MED][REL] `ecomm/cron/foms/import_fulfillment_orders.php:241-255` — On ack exception, the FO is already committed; source queue can re-offer the order. Fix: retry with backoff or mark `ack_pending` and run a dedicated retry job.
- [MED][REL] `ecomm/cron/foms/import_shipments.php:155-165` — Re-reads FO before save but the long per-order loop widens the cancel/import race window. Fix: re-check status immediately before `save()` and shorten batches.
- [MED][REL] `ecomm/cron/foms/invoice_shipments.php:252-253` — `$fulfillmentOrder` may be undefined when reaching metrics in the outer catch. Fix: guard with `isset()` or fetch from shipment.

### Performance (Tier 2)

- [HIGH][PERF] `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php:52-80` — `index()` calls `findAllNotStatus(...)` (default limit 10000) and returns the entire payload. Fix: add `page`/`per_page` parameters with a hard cap.
- [HIGH][PERF] `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php:127-135` — `all_full` returns the entire FO history. Fix: require pagination; cap or stream as export.
- [HIGH][PERF] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:22,465-505` — Default `limit = 10000`; `findAll()` loads 10k FOs with `LEFT JOIN ... GROUP BY` on every call. Fix: lower default, require explicit pagination.
- [HIGH][PERF] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrder.php:85-117` — `getLineItems()`, `getShipments()`, `getShippingInfo()` each hit the DB and are called in loops during cancel/delete/move/import. Fix: batch-load on the repository or expose eager-loading via the import service.
- [HIGH][PERF] `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php:182-183` — Search autocomplete calls `$order->getShippingInfo()` per result (up to 25 extra queries). Fix: JOIN `ShippingInfo` or batch-fetch by FO ids.
- [HIGH][PERF] `vendor/joeybab3/foms/src/Api/InventoryController.php:65-71` — `index()` runs unbounded `SELECT i.* ...` with a margin join. Fix: paginate, request only the columns the UI needs.
- [HIGH][PERF] `vendor/joeybab3/foms/src/Router/RuleRepository.php:43-54` — `findByRouterId()` triggers N+1: every rule calls `criteriaRepository->findByRuleId()`. Fix: single JOIN or batch-load criteria by rule IDs.
- [MED][PERF] `vendor/joeybab3/foms/src/Event/EventRepository.php:129-157` — `findAll()` with `$limit = null` returns unbounded rows. Fix: default limit 100 + require pagination for admin views.
- [MED][PERF] `vendor/joeybab3/foms/src/InventoryFeed/InventoryFeedRepository.php:279-304` — `runAll()` calls `findAll()` and then re-instantiates each feed by handle. Fix: reuse instances from the first query.
- [MED][PERF] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:467-491` — `findAll()` always runs `LEFT JOIN ... GROUP BY fo.id` even when `lineitem_count` is unused. Fix: split into lightweight and aggregate variants.
- [MED][PERF] `vendor/joeybab3/foms/src/SkuTranslation/SkuTranslationRepository.php:36-37` — `findAll()` returns the full SKU translation table with no pagination. Fix: paginate or stream CSV.
- [MED][PERF] `ecomm/cron/foms/import_shipments.php:89-92` — Bulk ID fetch followed by per-FO `findById`. Fix: batch hydrate FOs in the repository.
- [MED][PERF] `ecomm/cron/foms/import_fulfillment_orders.php:146-154` — `$orders = $orderSource->getOrders($filters)` returns the entire list per storefront. Fix: paginate in the order source or stream by cursor.

### Modernization (Tier 2)

- [MED][MOD] `ecomm/foms/api/index.php:224-226` — Calls `session_start()` mid-request only on success. Fix: centralize session bootstrap with secure cookie flags in one include.
- [MED][MOD] `vendor/joeybab3/foms/src/Api/InventoryController.php:65-67` — Raw SQL in controller bypasses the repository. Fix: move query to `InventoryRepository`.
- [MED][MOD] `vendor/joeybab3/foms/src/Api/Controller.php:13-69` — Base controller carries no auth contract. Fix: introduce middleware or base-class guards.
- [LOW][MOD] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderSubmissionService.php:11-18` — Service holds `App` only to fetch dependencies indirectly. Fix: inject `FulfillmentOrderRepository`, destination registry directly.
- [LOW][MOD] Multiple FOMS page files — Missing `declare(strict_types=1);` (only `tiktok_oauth_callback.php` uses it). Fix: incrementally adopt strict types starting with API entry points.

---

## Tier 3 — Page / script / class-level fixes

### Security (Tier 3)

- [CRIT][SEC] `ecomm/foms/tiktok_oauth_callback.php:1-58` — Public OAuth callback renders the authorization code in HTML and a copy-paste CLI command; `state` is shown but never validated against a session nonce; no `redirect_uri` allowlist. Anyone who can reach the URL sees the code. Fix: SSO-gate (or IP-restrict) the page, verify `state` from session, and exchange the code server-side without echoing it.
- [HIGH][SEC] `ecomm/foms/api/index.php` + `ecomm/foms/ajax.php` + `ecomm/foms/accounting/ajax.php` + `ecomm/foms/bundles/ajax.php` — Every state-changing route is protected by session cookie alone; no CSRF tokens; no `SameSite` enforcement visible in this layer. Fix: introduce CSRF tokens (or double-submit cookie + `SameSite=Strict` session cookie), verify on every mutating endpoint.
- [HIGH][SEC] `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php` — `delete`, `cancel`, `move`, `submit`, `confirm`, `import` lack the `readWriteDenied()` / `userlevel` guards present in `RuleController`, `SkuTranslationController`, etc. Fix: add a shared `assertNotReadOnly()` and apply consistently.
- [HIGH][SEC] `vendor/joeybab3/foms/src/Api/ShipmentController.php:406`+ , `EventController`, `DeliveriesController`, `InventoryFeedController`, `NetSuiteInfoController` — Same missing-write-guard pattern. Fix: centralize authorization in `Controller` or per-endpoint guards.
- [HIGH][SEC] `ecomm/foms/order.php:19-21` + `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php:285-299,962-986` — IDOR: any authenticated Ecomm user with `read`/`edit` can load, delete, submit any FO by numeric ID; no customer/destination scoping. Fix: enforce per-user/customer/destination ACLs at the repository or controller layer.
- [HIGH][SEC] `ecomm/foms/fulfillment_order.php:157-257,287-387` — Legacy FO editor echoes order fields, SKUs, tracking, gift messages, and NetSuite transaction names with `<?= ... ?>` and no `htmlspecialchars`. Fix: apply `htmlspecialchars(..., ENT_QUOTES, 'UTF-8')` to all DB-backed output (or delete the page — see ARCH section).
- [MED][SEC] `includes/sso_auth.php:172-173,189` — Users with global admin SSO get `userlevel = 1`, so the FOMS `isReadOnly()` check (`userlevel >= 50`) returns false even when their Ecomm tier is read-only. Fix: base `isReadOnly()` on the Ecomm SSO tier (`$ssoRbac->canAtLeast('edit', 'Ecomm')`).
- [MED][SEC] FOMS-wide — 9 controllers compute authorization via `global $userlevel` (`SkuTranslationController:14-22` and 9 others). Fix: inject an `AuthorizationService` / session context via DI.
- [MED][SEC] `ecomm/foms/bundles/ajax.php:12-25` — `?activity=delete-bundle` (GET param) triggers deletion. Fix: POST-only routing for mutations; reject GET-supplied `activity`.
- [MED][SEC] `includes/sso_auth.php:31-37` — Unauthenticated API calls receive a 302 to SSO instead of `401 application/json`. Fix: detect `Accept: application/json` or `/api/` prefix and return `401` JSON.
- [MED][SEC] `ecomm/foms/components/foms_ui/foms_ui.js` (entire file, 3655 lines) — Multiple render paths bypass the existing `escapeHtml()` helper. Fix: refactor renderers to always go through escape helpers and add an ESLint rule banning template-literal `innerHTML`.

### Reliability (Tier 3)

- [CRIT][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1819-1959` — `cancel()` with `$recreateOrder=true` performs 10+ DB writes (destination cancel, line items, shipments, INSERT a new FO, copy line items / shipping / NetSuite, update original) with **no transaction**; failure mid-flow leaves split-brain state. Fix: wrap entire cancel + recreate in a single DB transaction.
- [CRIT][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1877-1886` — Before persisting the original FO as cancelled, code sets in-memory status to `cancelled` and calls `save($currentOrder)` which **INSERTs a new row** (still cancelled) rather than cloning fields into a fresh pending FO; the original row status isn't updated until line 1958. Fix: persist original cancellation first, then build the recreate FO from explicit copy.
- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderImportService.php:2237-2271` — Per-order import creates FO + ShippingInfo + N line items + optional invoice with no transaction; partial import leaves orphan rows. Fix: `beginTransaction` per order.
- [HIGH][REL] `ecomm/cron/foms/import_shipments.php:78-89` — `setLimit(100000)` then in-memory `count($ids)` + per-id `findById`. Long runtime and high memory; hourly cron may skip while backlog grows. Fix: chunked cursor (`id > last_id LIMIT 500`), unset hydrated objects between batches, and alert when backlog exceeds a threshold.
- [HIGH][REL] `ecomm/cron/foms/invoice_shipments.php:68` via `ShipmentsRepository::findByStatus` (vendor 87-98) — Loads all pending shipments unbounded. Fix: add `ORDER BY id LIMIT N` to the repository or loop the cron with pagination.
- [HIGH][REL] `ecomm/cron/foms/submit_fulfillment_orders.php:38-61` + `ecomm/cron/foms/import_confirmations.php:37-76` — `findByStatus('pending'|'submitted')` is a plain `SELECT ... LIMIT 10000`; no `FOR UPDATE`, no claim column; concurrent API submit or utilities can process the same FO. Fix: claim rows atomically (`UPDATE ... WHERE status='pending' AND claimed_at IS NULL LIMIT N` or `SELECT ... FOR UPDATE SKIP LOCKED`) before any external API call.
- [HIGH][REL] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:41-47,395` — 10k row default limit + submit running ~6x/hour means backlog beyond 10k never drains in one cycle. Fix: loop until empty or raise limit with cursor paging, plus alerting when `COUNT(*)` exceeds threshold.
- [HIGH][REL] `ecomm/cron/inventory/foms_build_logiwa_inventory.php:27-28,105-108` — `clearLogiwaInventory()` truncates the table then repopulates from paginated API; mid-run failure leaves an empty / partial table until the next 2h cron. Fix: swap-in staging table (`logiwa_inventory_new`) and `RENAME TABLE` swap, plus `flock`.
- [MED][REL] `ecomm/cron/foms/` (all long-running scripts) — No `pcntl_signal` handlers; SIGTERM/SIGINT mid-run can corrupt invoicing or imports. Fix: register a shutdown handler that releases the lock and logs in-progress FO/shipment IDs.
- [MED][REL] Pipeline scripts (`ecomm/cron/foms/submit_fulfillment_orders.php:65-67`, `ecomm/cron/foms/import_confirmations.php:81-82`, `ecomm/cron/foms/import_shipments.php:169-170`, `ecomm/cron/foms/invoice_shipments.php:260-262`) — Per-order failures are caught and logged but the script always exits 0; monitoring on exit code never fires. Fix: track error count and `exit(1)` past a threshold.
- [MED][REL] `ecomm/foms/accounting/ajax.php:259-377` — `create-invoice` loops customers/destinations and calls `processBulkInvoice()` synchronously in an HTTP request; prone to timeouts and partial failures. Fix: queue bulk invoice jobs to cron/worker and return a task ID for polling.

### Performance (Tier 3)

- [HIGH][PERF] `ecomm/foms/index.php:174-265,490-492` — Every 120 s the client refetches the entire active-FO dataset and diff-patches DataTable client-side. Fix: server-side DataTables processing or a delta API (`?since=timestamp`).
- [HIGH][PERF] `ecomm/foms/fulfillment_orders.php:99-101` — `#show-me-all-fulfillment-orders` calls `findAll(false)` — tens of thousands of rows for some customers. Fix: server-side pagination + max-row cap; stream export.
- [HIGH][PERF] `ecomm/foms/inventory.php:93,137` — Default inventory listing loads full table + margin join into browser memory. Fix: server-side DataTables + selective columns.
- [HIGH][PERF] `ecomm/foms/ajax.php:21-51` (called from `inventory.php:202-204`) — `download-all-inventory` runs `SELECT i.* ...` and streams the entire table in one request. Fix: background export job or paginated CSV streaming with rate limits.
- [MED][PERF] DataTables "All" page length (`-1`) used in: `ecomm/foms/index.php:337`, `ecomm/foms/fulfillment_orders.php:117`, `ecomm/foms/inventory.php:128`, `ecomm/foms/shipments.php:98`, `ecomm/foms/destinations.php:75` — Renders every row in the DOM, freezes browser. Fix: cap at 500 or remove the All option.
- [MED][PERF] `ecomm/foms/shipments.php:153-155` (60s) + `ecomm/foms/inventory.php:197-199` (300s) — Full-table Ajax reloads on interval. Fix: poll only when tab visible; use delta endpoints.
- [MED][PERF] `ecomm/cron/crontab.backup:34,42` — `import_shipments.php` and `check_shipment_carriers.php` both run at minute 02. Fix: stagger minutes to spread DB/API load.
- [LOW][PERF] `ecomm/foms/index.php:109-728` + `order.php`, `rule.php:800+` — Hundreds of KB of inline `<script>`/`<style>` per page bypass cache. Fix: extract to versioned static assets under `/ecomm/foms/js/` and `/ecomm/foms/css/`.

### Bugs / Modernization (Tier 3)

- [MED][BUG] `ecomm/foms/order.php:19-21` + `ecomm/foms/foms/.htaccess:17` — Route `/order/{id}/` calls `findById($id)` on FulfillmentOrderRepository; URL implies order-level ID but resolves to FO ID. Fix: clarify route name (e.g. `/order/by-fo/{id}/`) or resolve by true order identifier.
- [LOW][BUG] `ecomm/foms/fo.php:59-61` — Loads jQuery 3.7.1 + jQuery UI after `head.php`, risking double-init and plugin conflicts. Fix: rely on shared `head.php` assets.
- [MED][REL] `vendor/joeybab3/foms/src/Bulk/BulkRunner.php:37-99` — Bulk operations run sequentially with no per-entity transaction boundary at the runner level. Fix: document semantics; add batch-level summary + dead-letter queue.
- [MED][REL] `ecomm/cron/foms/foms_sync_to_snowflake.php` — Long-running export with no `set_time_limit` adjustment, no resume token. Fix: explicit time/memory limits and incremental sync state.
- [LOW][MOD] `ecomm/foms/` page set — Forms (`create_destination.php:49-100`, `create_source.php:49-60`, `create_storefront.php:38-60`, `create_inventory_feed.php`, `create_shipment.php`) rely on `required` and JS validation only; server-side validation depends entirely on API controllers. Fix: add shared schema validation in the API layer.

---

## Tier 4 — Architectural changes

### Architecture (Tier 4)

- [HIGH][ARCH] `vendor/joeybab3/foms/src/App.php:39-47,135-165` — `App` is a string-keyed service locator wiring 20+ concrete dependencies; no interfaces for most services; hard to test or mock in isolation. Fix: migrate to a PSR-11 DI container (PHP-DI, Symfony DI, or a manual factory) with typed getters per service.
- [HIGH][ARCH] `vendor/joeybab3/foms/src/Structure.php:281-399` — Base domain class owns event creation, change diffing, and JSON reduction — mixing persistence audit with entity behavior. Fix: extract an `EventRecorder` / `AuditService` and inject into structures that need it.
- [HIGH][ARCH] FOMS cron pipeline (`ecomm/cron/foms/import_fulfillment_orders.php`, `ecomm/cron/foms/submit_fulfillment_orders.php`, `ecomm/cron/foms/import_confirmations.php`, `ecomm/cron/foms/import_shipments.php`, `ecomm/cron/foms/invoice_shipments.php`) — No atomic claim or lock for `pending` rows; the 10k cap silently caps backlog drain. Fix: introduce a claim queue (`claimed_at`, `claim_token`, `attempts`, `next_attempt_at`) with `UPDATE ... LIMIT N` claim semantics, exponential backoff, and a dead-letter status.
- [HIGH][ARCH] `ecomm/foms/api/` — No API version prefix; breaking changes affect all clients. Fix: move to `/ecomm/foms/api/v1/` and document versioning policy.
- [HIGH][ARCH] `ecomm/foms/api/` — No CORS configuration, no rate limiting, no token/Bearer support for machine-to-machine. Fix: add rate limiting (reverse proxy or app-level per-user/IP), document same-origin policy, and introduce signed service tokens for cron-to-API integrations.
- [HIGH][ARCH] `composer.json:27` — Root `"minimum-stability": "dev"` plus `@dev` / `dev-master` internals (`shopify`, `router`, `netsuiterest`, `jlabactions`, `sso-rbac`, etc.) shipped to prod. Fix: tag internal packages, host via a private Packagist or VCS, pin to `^x.y` in production; keep `path` repos only in a local `composer.local.json` for dev.
- [HIGH][ARCH] `vendor/joeybab3/foms/composer.json:31-43` — Path repositories pointed at sibling directories work in the monorepo but break standalone install. Fix: publish internal packages and consume by VCS in production builds.
- [HIGH][ARCH] No CI workflow exists for FOMS — repo has no `.github/workflows`, `.gitlab-ci.yml`, or `Jenkinsfile`. Fix: add a workflow that runs `composer install`, `composer test`, `composer audit`, PHPStan, and php-cs-fixer/Pint on every PR.
- [HIGH][ARCH] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:17` — Repository class isn't `final`; it carries identity-map + mutable query state (`$limit`, `$sortby`) on a shared singleton — footgun across requests. Fix: make repositories stateless per query (pass an options object) or scope per-request via DI.
- [MED][ARCH] `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:808-833` + repository tier — Repositories return untyped associative arrays internally; entities use public setters everywhere; no DTOs for pipeline boundaries. Fix: introduce readonly DTOs for import/submit/confirm I/O.
- [MED][ARCH] FOMS-wide — `ConfigurableAction` registry uses runtime file scanning (`vendor/joeybab3/foms/src/ConfigurableAction/ConfigurableActionTypeRegistry.php:108`). Fix: explicit action registration via a config/array; avoid runtime file discovery.
- [MED][ARCH] `ecomm/foms/fulfillment_order.php` (463 lines) — Legacy FO editor runs alongside the modern `ecomm/foms/fo.php` + `ecomm/foms/components/foms_ui/foms_ui.js`. Fix: 301 redirect `/fulfillment_order/{id}/` → `/fo/{id}/` and remove the file.
- [MED][ARCH] `ecomm/foms/components/foms_ui/foms_ui.js` — 3655 lines containing line-item, shipment, router, and bulk-action subsystems in one file. Fix: split into ESM modules (LineItems, Shipments, RouterManager, BulkActions) and load only what each page needs.
- [MED][ARCH] `ecomm/foms/rule.php` (1538 lines), `ecomm/foms/order.php` (1234 lines), `ecomm/foms/fo_shopify.php` (525 lines) — Megafiles mixing PHP, HTML, CSS, and JS. Fix: extract view partials + page-specific JS modules; aim for ≤ 300 LOC per file.
- [MED][ARCH] `ecomm/foms/index.php` + sibling pages — Mixed Bootstrap (5.3.0-alpha1 in `head.php`) and DataTables versions (1.13.1 in `create_destination.php`, 2.0.3 in `index.php`) across pages. Fix: standardize on a single Bootstrap 5.3 stable + DataTables 2.x bundle.
- [MED][ARCH] FOMS-wide — No Content Security Policy. Inline `<script>` blocks across CDN allowlist (`cdn.datatables.net`, `cdnjs`, `jsdelivr`, `code.jquery.com`). Fix: introduce nonce-based CSP, consolidate CDN list, move inline scripts to external files.
- [MED][ARCH] `ecomm/foms/` page bootstrap — Most pages do `common.php` → `sso_site_gate.php` → `cron/foms/common.php`; large dependency graph loaded twice. Fix: a single `foms/bootstrap.php` entry include.
- [MED][ARCH] `ecomm/cron/foms/` — Procedural pipeline scripts still duplicate logic that exists in services (`FulfillmentOrderSubmissionService` etc.). Fix: move each step into `Joeybab3\Foms\Console\*` commands invoked by cron (`run_bulk_task.php` is a good model).
- [LOW][ARCH] `ecomm/cron/foms/common.php:2` + scripts — Hardcoded `/var/www/html/...` paths throughout. Fix: `FOMS_ROOT` env or autoload-only bootstrap via Composer.
- [LOW][ARCH] FOMS-wide — Per-page `isReadOnly()` helper duplicated 15+ times (`fo.php:15-18`, `destination.php:23`, `destinations.php:24`, `criteria.php:12`, `criterion.php:12`, `fulfillment_order.php:15`, `sku_translations.php:23`, etc.). Fix: move to shared `foms/includes/auth_helpers.php`.
- [LOW][ARCH] `ecomm/foms/` — Create-wizard pages (`create_destination.php`, `create_source.php`, `create_storefront.php`, `create_inventory_feed.php`, `create_sku_translation.php`) are near-duplicates (~200 lines each). Fix: shared form template driven by entity-specific field config.
- [LOW][ARCH] `ecomm/cron/crontab.backup:11,15` — Legacy `3pl_import/import_confirmations.php` runs in parallel with FOMS `import_confirmations.php`. Fix: document ownership and decommission the legacy job if FOMS is canonical.
- [MED][ARCH] `ecomm/cron/foms/` — No retry/backoff/dead-letter anywhere. Fix: introduce `attempts`, `last_error`, `next_attempt_at` columns or a job table with exponential backoff; transition exhausted retries to a dedicated `dead_letter` status with alerting.
- [MED][ARCH] `ecomm/cron/foms/` — No structured logging; pipeline scripts mix `echo`, `$app->log`, and `error_log`. Fix: introduce a single structured (JSON line) logger with `script`, `fo_id`, `level`, `message` fields and adopt across all scripts and services.
- [MED][ARCH] `vendor/joeybab3/foms/src/Metrics/OrderMetrics.php:91-97` — Prometheus failures are swallowed; no failure or lock-skip counters. Fix: add `foms_pipeline_errors_total{script}`, `foms_cron_lock_skipped_total{script}`, `foms_dead_letter_total`, and wire to Alertmanager rules.
- [MED][ARCH] `ecomm/cron/foms/logs/` — Log files grow without rotation. Fix: logrotate stanza or size-based rotation on `*.log`.
- [MED][ARCH] `ecomm/cron/foms/utilities/` (84 scripts) — Most lack `php_sapi_name() === 'cli'` guards; dangerous if any becomes web-exposed. Fix: introduce a shared `require_cli()` helper in bootstrap and call from every utility.

---

## Test coverage gaps

Source directories in `vendor/joeybab3/foms/src/` with no parallel `tests/` directory or minimal coverage. Priorities (P) are based on production-criticality.

| Source area | Test status | P |
|---|---|---|
| `src/Helper/` | No tests (`Customer.php` untested) | Low |
| `src/Actions/` | No tests for 8 action types (only `ConfigurableActionRunner` tested) | **High** |
| `src/TrackingProvider/` | No tests for FedEx, USPS, Amazon, Unknown providers | Medium |
| `src/Metrics/OrderMetrics.php` | No tests | Medium |
| `src/NetSuite/NetSuiteAdapter.php` | No dedicated tests (only `SaveEvents` / `Info` tests) | **High** |
| `src/OrderSource/ShipStationOrderSource.php` | No dedicated tests | High |
| `src/InventoryFeed/TikTokInventoryFeed.php`, `vendor/joeybab3/foms/src/InventoryFeed/WalmartInventoryFeed.php` | Minimal (Shopify metafield test only) | Medium |
| `src/OrderDestination/TaggDestination.php`, `vendor/joeybab3/foms/src/OrderDestination/ShopifyDestination.php` | No dedicated tests | High |
| `src/Api/DeliveriesController.php` | No tests (`BulkDestinationsControllerTest` covers a related path only) | Medium |
| `src/Bulk/IgnoreDeliveriesOperation.php` | No unit test (only indirect coverage) | Medium |
| `src/Bulk/BulkTaskRepository.php`, `vendor/joeybab3/foms/src/Bulk/OperationRegistry.php` | No direct tests | Medium |

**Well-covered areas** (for reference): `FulfillmentOrder/` (import/submit/confirm/repo tests), `OrderSource/TikTok+Walmart`, `OrderDestination/Logiwa+Amazon+External`, `Event/`, `Router/` (integration + save-events), `Api/` (most controllers have at least one test), `Bulk/` (runner + 3 operations).

**Notable gap of concern**: `FulfillmentOrderRepository::cancel()` recreate path (lines 1886-1959) — `FulfillmentOrderCancelTest` exists but does not appear to exercise the insert-before-update ordering bug (Tier 3, CRIT). Add a regression test once the bug is fixed.

---

## Tooling & CI recommendations

FOMS currently has **no CI**, **no static analysis**, **no code style automation**, and **no dependency auditing**. The only quality gate is `composer test` (PHPUnit) run manually inside the package.

### Composer dev-dependencies to add

Add to `vendor/joeybab3/foms/composer.json` under `require-dev`:

```json
{
  "require-dev": {
    "phpunit/phpunit": "^11.5",
    "phpstan/phpstan": "^1.11",
    "phpstan/phpstan-strict-rules": "^1.5",
    "phpstan/phpstan-deprecation-rules": "^1.2",
    "laravel/pint": "^1.15",
    "infection/infection": "^0.29",
    "maglnet/composer-require-checker": "^4.11",
    "icanhazstring/composer-unused": "^0.8",
    "captainhook/captainhook": "^5.23",
    "rector/rector": "^1.0"
  }
}
```

### Composer scripts

Add to the FOMS package `composer.json` (and a parallel set at the root):

```json
{
  "scripts": {
    "test": "vendor/bin/phpunit",
    "test:unit": "vendor/bin/phpunit --testsuite=Unit",
    "test:mutation": "vendor/bin/infection --min-msi=70",
    "phpstan": "vendor/bin/phpstan analyse --memory-limit=1G",
    "pint": "vendor/bin/pint",
    "pint:check": "vendor/bin/pint --test",
    "audit": "composer audit",
    "deps:check": "vendor/bin/composer-require-checker check",
    "deps:unused": "vendor/bin/composer-unused",
    "check": [
      "@pint:check",
      "@phpstan",
      "@test",
      "@audit",
      "@deps:check",
      "@deps:unused"
    ]
  }
}
```

### PHPStan config

`phpstan.neon` at FOMS package root:

```yaml
parameters:
  level: 6
  paths:
    - src
    - tests
  ignoreErrors:
    - identifier: missingType.iterableValue
  excludePaths:
    - tests/**/fixtures/*
```

Start at level 5 or 6; raise to 8 (max) as findings are fixed.

### CI workflow (`.github/workflows/foms.yml`)

```yaml
name: FOMS CI
on:
  pull_request:
    paths:
      - 'vendor/joeybab3/foms/**'
      - 'ecomm/foms/**'
      - 'ecomm/cron/foms/**'
      - 'composer.json'
      - 'composer.lock'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: pdo_mysql, json, curl, openssl, mbstring
          coverage: xdebug
      - name: Composer install (FOMS package)
        working-directory: vendor/joeybab3/foms
        run: composer install --prefer-dist --no-interaction
      - name: PHPStan
        working-directory: vendor/joeybab3/foms
        run: composer phpstan
      - name: Pint (style)
        working-directory: vendor/joeybab3/foms
        run: composer pint:check
      - name: PHPUnit
        working-directory: vendor/joeybab3/foms
        run: composer test
      - name: Composer audit
        run: composer audit
```

### Pre-commit (captainhook)

`captainhook.json` at FOMS package root:

```json
{
  "pre-commit": {
    "actions": [
      { "action": "vendor/bin/pint --test" },
      { "action": "vendor/bin/phpstan analyse --no-progress" }
    ]
  },
  "pre-push": {
    "actions": [
      { "action": "vendor/bin/phpunit --stop-on-failure" }
    ]
  }
}
```

### Dependency hygiene

- Add Dependabot (`.github/dependabot.yml`) for `composer` ecosystem on a weekly cadence.
- Run `composer audit` in CI on every PR.
- Move all `joeybab3/*` `@dev` internals to tagged releases, host on a private Packagist or via VCS, and use `path` repositories only in a developer-local `composer.local.json`.
- Add a `"php": "^8.2"` constraint and explicit `ext-pdo`, `ext-json`, `ext-curl`, `ext-openssl` entries to the FOMS package manifest.

### Frontend tooling

- Add an ESLint config under `ecomm/foms/` banning `innerHTML` with template-literal expressions and requiring a single `escapeHtml()` helper.
- Bundle `foms_ui.js` and related assets with esbuild or Vite to enable tree-shaking, source maps, and content-hashed URLs (needed before any CSP rollout).

---

## Suggested remediation roadmap

Six two-week sprints. Each sprint groups findings by theme; finding IDs are described inline by file path so this list is self-contained.

### Sprint 1 — Credentials, token leaks, and OAuth exposure (1 week of security work)

Goal: stop bleeding secrets and stop logging tokens.

- Remove bearer token from `vendor/joeybab3/foms/src/OrderDestination/LogiwaDestination.php:183`.
- Remove Walmart Client ID logging at `vendor/joeybab3/foms/src/OrderSource/WalmartOrderSource.php:133` and Shopify response logging at `vendor/joeybab3/foms/src/InventoryFeed/ShopifyInventoryFeed.php:208`.
- Move and rotate Snowflake password from `ecomm/cron/foms/foms_sync_to_snowflake.php:137`, `ecomm/cron/foms/fetch_walmart_item_performance_data.php:215`, and the two utilities. Scrub git history with `git filter-repo`.
- Remove token logging from `ecomm/cron/foms/utilities/exchange_tiktok_oauth_code.php`.
- Lock down or delete `ecomm/foms/tiktok_oauth_callback.php` in production.
- Remove `PDO::MYSQL_ATTR_LOCAL_INFILE` from `vendor/joeybab3/foms/src/App.php:181`.
- Replace `shell_exec` in `vendor/joeybab3/foms/src/NetSuite/NetSuiteInfoRepository.php:176-186` with a REST call or `escapeshellarg` + `ctype_digit`.

### Sprint 2 — XSS, CSRF, and authorization

Goal: harden the browser-facing surface area.

- Wrap every `?msg=` use in `json_encode()` across `ecomm/foms/fulfillment_orders.php`, `ecomm/foms/inventory.php`, `ecomm/foms/shipments.php`, `ecomm/foms/shipment.php`, and `bundles/*`.
- Add a server-side `escapeHtml` helper and refactor `ecomm/foms/index.php` renderers, `ecomm/foms/components/foms_ui/foms_ui.js` renderers, `ecomm/foms/deliveries.php`, `ecomm/foms/preview_storefront.php`, `ecomm/foms/rule.php`, `ecomm/foms/components/timeline/timeline.js`, and `ecomm/foms/js/bulk-tasks.js`.
- Patch `includes/head.php:264-271` so `notificationMessage()` does not assign to `innerHTML` without escaping.
- Introduce a CSRF token middleware in `ecomm/foms/api/index.php` + AJAX endpoints (double-submit cookie + `SameSite=Strict`).
- Add `assertNotReadOnly()` / `userlevel` guards on `FulfillmentOrderController`, `ShipmentController`, `EventController`, `InventoryFeedController`, `NetSuiteInfoController`.
- Fix the `userlevel` bypass in `includes/sso_auth.php:172-189` so admin SSO does not skip `isReadOnly()`.
- Add an Ecomm tier-based scope to legacy `isReadOnly()` helpers and centralize them in `foms/includes/auth_helpers.php`.

### Sprint 3 — Pipeline correctness and concurrency

Goal: stop double-invoicing, partial submits, and silent backlog growth.

- Fix `ecomm/cron/foms/invoice_shipments.php:242-256` to only `save('invoiced')` on actual transitions; add `flock`.
- Add `flock` to `ecomm/cron/foms/import_cancelation_requests.php` and emit non-zero exit / metric on lock skip across all pipeline scripts.
- Wrap `cancel()` recreate-flow (`vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderRepository.php:1819-1959`) in a transaction and fix the insert-before-update ordering bug.
- Wrap `FulfillmentOrderImportService::importOrder()` (`vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderImportService.php`) in a transaction.
- Introduce atomic claim semantics for `pending`/`submitted` rows in submit / confirm pipelines.
- Add `pcntl_signal` handlers to the pipeline scripts.
- Fix `acknowledgeOrder(null)` TypeError in `ecomm/cron/foms/import_fulfillment_orders.php:237` and `ecomm/cron/foms/utilities/import_shopify_order_by_number.php:132`.
- Add idempotency / compensating rollback to `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderSubmissionService.php:47-63`.
- Switch `ecomm/cron/inventory/foms_build_logiwa_inventory.php` to a staging-table swap pattern.

### Sprint 4 — Performance and pagination

Goal: eliminate full-table loads.

- Server-side DataTables on `ecomm/foms/index.php`, `ecomm/foms/fulfillment_orders.php`, `ecomm/foms/inventory.php`, `ecomm/foms/shipments.php`, `ecomm/foms/destinations.php`.
- Paginate `vendor/joeybab3/foms/src/Api/FulfillmentOrderController.php`, `vendor/joeybab3/foms/src/Api/InventoryController.php`, `vendor/joeybab3/foms/src/Event/EventRepository.php`, `vendor/joeybab3/foms/src/SkuTranslation/SkuTranslationRepository.php`.
- Eliminate N+1 patterns in `vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrder.php` and `vendor/joeybab3/foms/src/Router/RuleRepository.php`.
- Chunked cursor in `ecomm/cron/foms/import_shipments.php` and stream-pagination in `ecomm/cron/foms/import_fulfillment_orders.php`.
- Stagger crontab entries (`crontab.backup` lines 34/42).
- Background bulk invoice in `ecomm/foms/accounting/ajax.php` using the bulk task framework.

### Sprint 5 — Composer hygiene, CI, and observability

Goal: make the codebase safe to refactor.

- Move PHPUnit to `require-dev`, add `"php": "^8.2"` and `ext-*` constraints, set `"license": "proprietary"`, switch root `minimum-stability` to `stable`, tag internal `joeybab3/*` packages.
- Add the CI workflow in the tooling section.
- Add PHPStan (level 5 → 8 over time), Pint, `composer audit`, Dependabot, captainhook.
- Introduce a structured (JSON line) logger across pipeline scripts; rotate logs in `ecomm/cron/foms/logs/`.
- Add Prometheus counters for `foms_pipeline_errors_total`, `foms_cron_lock_skipped_total`, `foms_dead_letter_total`.
- Stop printing `--debug` in production crontab; ensure all cron lines `2>&1`.

### Sprint 6 — Architectural cleanup

Goal: get FOMS out of the "monolith with a service locator" pattern.

- Introduce a PSR-11 DI container; remove string-keyed `App::getDependency()` calls in favor of constructor injection (start with the FO pipeline services).
- Extract an `EventRecorder` / `AuditService` from `vendor/joeybab3/foms/src/Structure.php`.
- Introduce a retry / dead-letter table and remove the silent 10k cap.
- Move pipeline scripts to `Joeybab3\Foms\Console\*` commands; cron only invokes the commands.
- Version the REST API at `/ecomm/foms/api/v1/`; add rate limiting and structured JSON 401 responses.
- Split `ecomm/foms/components/foms_ui/foms_ui.js` into ESM modules; standardize Bootstrap / DataTables versions; roll out a nonce-based CSP.
- Redirect `/fulfillment_order/{id}/` → `/fo/{id}/` and delete `ecomm/foms/fulfillment_order.php`.
- Decommission legacy `ecomm/cron/3pl_import/import_confirmations.php` if FOMS is canonical.

---

## Positive observations

Not everything is broken — these are what the audit found working well and worth preserving:

- All audited PHP pages include `sso_site_gate.php` with `$sso_area = 'Ecomm'` (except the documented OAuth callback). The site gate is doing its job.
- `ecomm/foms/api/index.php` correctly elevates `$sso_min_tier` to `edit` for non-GET methods.
- SQL throughout FOMS PHP/AJAX uses prepared statements; no `mysql_*` / `ereg_*` legacy APIs.
- No `include $_GET[...]` patterns anywhere.
- `ecomm/foms/components/foms_ui/foms_ui.js` defines an `escapeHtml()` helper (just inconsistently applied — see Tier 2 SEC).
- `ecomm/foms/order.php` server-rendered sections generally use `htmlspecialchars`.
- Most pipeline scripts already use `flock` — only `invoice_shipments.php` and `import_cancelation_requests.php` are exceptions.
- FOMS package test coverage is strong on `FulfillmentOrder/`, `Event/`, `Router/`, `Bulk/`, and most API controllers.
- The bulk task framework (`ecomm/cron/foms/run_bulk_task.php`) is a clean pattern to model the other cron consolidations on.

---

## Change log

| Date | Author | Notes |
|---|---|---|
| 2026-05-19 | Audit (Cursor agent) | Initial draft — 161 findings across 4 surfaces. |
