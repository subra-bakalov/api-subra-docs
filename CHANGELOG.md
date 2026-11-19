# Промени за клиенти

[English](CHANGELOG.en.md)

Тук са промените по Subra Products API, които засягат интеграцията. Най-новото е отгоре. Пълното описание е в [ръководството](README.md).

## 2026-11-19

Одобреният каталог вече има две нива.

Списъкът `GET /export/approved-products` и `GET /api/v1/approved-products` връща `"view": "summary"`. В него са цената, наличността, снимките и идентификаторите. Общият ключ чете целия каталог от `/export/approved-products`. Клиентският токен на `/api/v1/approved-products` вижда само своите одобрени продукти. Полетата `description`, `leaflet`, `search_words`, `meta_title`, `meta_keywords` и `meta_description` се четат от детайла:

```http
GET /api/v1/approved-products/{id}
GET /export/approved-products/{id}
```

Детайлът връща `"view": "full"`.

За повторяемо теглене:

- Когато заявката носи `updated_since`, `removed` съдържа номерата, които вече не са в каталога. Магазинът ги маха. Пълното теглене без `updated_since` връща празен `removed`.
- Повторно подаване на заявка със същия `Idempotency-Key` и същото тяло връща първата заявка и не създава втора.
- Отговорът носи `X-RateLimit-Limit` и `X-RateLimit-Remaining`. При `429` тялото е `{ "ok": false, "error", "limit", "window" }` и включва `Retry-After` и `X-Request-ID`.
- Всеки отговор на каталога носи `X-Request-ID`. При грешка цитирайте този номер.
- Общият ключ има лимит 120 заявки на минута и 3000 на час. Над лимита отговорът е `429` с `Retry-After`. Клиентският токен ползва лимита, зададен за него.
- `updated_since` връща продуктите, променени след последния успешен час.
- Отговорът носи `ETag`. Следващата заявка със същия `If-None-Match` получава `304` и празно тяло, когато резултатът не се е променил.
- Отговорът има `Cache-Control: private, max-age=45`.

Ако четете `description` от списъка, минете към адреса за един продукт.

## 2026-10-29

Ръководството описва текущия договор за партньори:

- Поръчките се четат от `GET https://subra.bg/api/partner/orders` с право `orders:read`.
- Продуктът има поле `brand`.
- Одобреният каталог приема `stock`, `updated_since`, `cursor` и `partner_tag`.
