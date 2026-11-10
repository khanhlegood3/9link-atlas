# 9Link — White Paper

**Link Intelligence Platform · All in One for Link**

| | |
|---|---|
| Phiên bản | 1.1 |
| Ngày viết | 2026-11-08 |
| Viết từ | Cây mã `main` @ `b559c68` (phiên bản production `8667e37`, deploy 2026-09-08) |
| Cập nhật 1.1 | 2026-11-10 · cây `main` @ `4afa314` · thêm §11.5 (so sánh trung thực) · §12.6 (white-label) · §12.7 (quyết định còn treo) · sửa ba con số trôi ở §14 và §15.2 |
| Bản tiếng Anh | `WHITEPAPER.en.md` — **đang lệch bản này**, tự khai ở dòng đầu |
| Bản công khai | Đăng ở repo công khai `khanhlegood3/9link-atlas` dưới tên `WHITEPAPER.md`, **lược §12.2–§12.3**, sinh bằng `scripts/redact-whitepaper.mjs` — không chép tay |
| Kiến trúc chi tiết | [`docs/c4/`](c4/README.md) — mô hình C4 bốn mức |
| Giấy phép | Core **AGPLv3** (`LICENSE`) · `/ee` **commercial** (`ee/LICENSE`) |

> **Cách đọc tài liệu này.** Đây là white paper của một sản phẩm **đã có mã chạy được**, không phải
> của một ý tưởng. Vì vậy nó tuân theo đúng kỷ luật mà repo này áp cho tài liệu của mình: **không
> nói rộng hơn mã**. Mục 12 liệt kê những gì *chưa* làm — và đó là mục nên đọc trước nếu bạn đang
> cân nhắc một cam kết thương mại với chúng tôi.

---

## Mục lục

1. [Tóm tắt điều hành](#1-tóm-tắt-điều-hành)
2. [Vấn đề: năm công cụ cho một đường link](#2-vấn-đề-năm-công-cụ-cho-một-đường-link)
3. [Luận điểm: một Link, một lớp Event](#3-luận-điểm-một-link-một-lớp-event)
4. [Sản phẩm: chín trụ trên cùng một URL](#4-sản-phẩm-chín-trụ-trên-cùng-một-url)
5. [Kiến trúc: bốn mặt phẳng và một đường nóng](#5-kiến-trúc-bốn-mặt-phẳng-và-một-đường-nóng)
6. [Mô hình kinh doanh: pivot 2026-08-25](#6-mô-hình-kinh-doanh-pivot-2026-08-25)
7. [Kinh tế học của cổng quảng cáo](#7-kinh-tế-học-của-cổng-quảng-cáo)
8. [Chủ quyền dữ liệu: wedge MENA](#8-chủ-quyền-dữ-liệu-wedge-mena)
9. [Niềm tin: chống lạm dụng là rủi ro tồn vong](#9-niềm-tin-chống-lạm-dụng-là-rủi-ro-tồn-vong)
10. [Open-core: seam là một ràng buộc pháp lý](#10-open-core-seam-là-một-ràng-buộc-pháp-lý)
11. [Thị trường và lộ trình](#11-thị-trường-và-lộ-trình)
12. [Những gì chưa làm — nói thẳng](#12-những-gì-chưa-làm--nói-thẳng)
13. [Rủi ro và cách chúng được quản trị](#13-rủi-ro-và-cách-chúng-được-quản-trị)
14. [Bằng chứng kỹ thuật](#14-bằng-chứng-kỹ-thuật)
15. [Phụ lục](#15-phụ-lục)

---

## 1. Tóm tắt điều hành

**9Link là một nền tảng nguồn mở hợp nhất trọn vòng đời của một đường link** — tạo, làm giàu, định
tuyến, phân phối, đo lường, tối ưu, kiếm tiền, quản trị, tích hợp — trên **một URL thương hiệu duy
nhất** và **một lớp dữ liệu sự kiện duy nhất**.

Nó thay thế một chồng năm công cụ rời mà mọi đội marketing đang trả tiền và đang ghép tay:

```
Bitly  +  Branch  +  một QR tool  +  GA4/UTM  +  Linktree
(short)   (deeplink)  (QR động)     (attribution)  (bio)
```

**Ba luận điểm chính:**

1. **Lợi thế cạnh tranh không phải tính năng — nó là lớp dữ liệu.** QR (trụ 2), deep link (trụ 3),
   lượt xem bio (trụ 4), lượt hiển thị quảng cáo (trụ 7) **đều sinh ra cùng một loại `Event`**, đổ
   về **cùng một bảng** trong ClickHouse. Đó là điều kiện để attribution liền mạch xuyên điểm chạm —
   và là thứ một chồng công cụ rời **không làm được về mặt cấu trúc**, bất kể từng công cụ tốt đến
   đâu. Chúng tôi gọi đây là *moat*, và nó được bảo vệ bằng một bất biến có cổng máy gác:
   *cấm tạo data-store song song*.

2. **Cloud miễn phí và không giới hạn; tiền đến từ chính lưu lượng.** Từ pivot 2026-08-25, 9Link
   **không bán dung lượng**. Số link · custom domain · bio page · analytics · lưu lượng — tất cả
   không giới hạn. Nguồn thu là **quảng cáo trên cổng của link** (9Link **là** mạng quảng cáo),
   chia lại cho chủ link, giữ phần chênh; cộng take-rate trên GMV affiliate; cộng giấy phép `/ee`
   doanh nghiệp. Đây là một mô hình mà **lợi ích của nền tảng và của người dùng thẳng hàng theo
   cùng một con số**: lưu lượng thật.

3. **Chủ quyền dữ liệu là điểm bán, không phải tính năng phụ.** RTL/Arabic hạng nhất, WhatsApp
   routing, và **data residency ba mức** (nhãn vùng → Event theo cụm → PII/Postgres theo cụm) với
   luật **fail-closed**: không phân giải được vùng thì **từ chối ghi**, không bao giờ rơi về cụm
   mặc định. Đây là thứ Bitly (đóng) không có, và là cửa vào enterprise/government vùng Vịnh.

**Trạng thái:** 45 mốc đã đóng, **đủ 9/9 trụ sống**, production đang chạy. Toàn bộ Core là AGPLv3
và self-host được bằng `docker compose`.

**Điều quan trọng nhất cần biết trước khi đọc tiếp:** chuỗi tiền quảng cáo hiện **inert trên
production theo thiết kế** — bảng giá (`ad_rate_cards`) có **0 hàng**, nghĩa là
`currentRates() == null`, nghĩa là cổng kiếm tiền **không tự bật** cho link mới và **không một dòng
sổ nào** được sinh ra. Đây không phải nợ kỹ thuật; đó là chiều fail đúng: *thiếu cấu hình ⇒ không
hứa tiền với ai*. Chi tiết ở §7.4 và §12.

---

## 2. Vấn đề: năm công cụ cho một đường link

### 2.1 Cái giá của việc ghép tay

Một đội marketing chạy một chiến dịch QR-to-app ở một thị trường mới nổi hôm nay phải:

| Bước | Công cụ | Dữ liệu sinh ra | Nằm ở đâu |
|---|---|---|---|
| Rút gọn + brand domain | Bitly / shortener | click | SaaS A |
| Deep link + deferred deep link | Branch / AppsFlyer | install, open | SaaS B |
| QR động có brand | một QR tool | scan | SaaS C |
| UTM + báo cáo | GA4 | session, conversion | SaaS D |
| Link-in-bio | Linktree | view, block click | SaaS E |

Năm hệ thống, năm định nghĩa "một lượt", năm bảng điều khiển, năm hoá đơn. Và **không một hệ nào
trả lời được câu hỏi duy nhất quan trọng**: *"chiến dịch này, tính từ lúc ai đó quét mã QR tới lúc
họ mua hàng, đã tạo ra bao nhiêu tiền — và link nào chịu trách nhiệm?"*

Câu hỏi đó không trả lời được **không phải vì thiếu tính năng**. Nó không trả lời được vì câu trả
lời đòi **join xuyên năm kho dữ liệu của năm nhà cung cấp**, mà mỗi kho có một định danh riêng, một
cửa sổ attribution riêng, và một định nghĩa "unique" riêng.

### 2.2 Ba hệ quả có thể định giá được

1. **Chi phí trực tiếp** — năm subscription, và ở quy mô agency là năm subscription **nhân số khách**.
2. **Chi phí quyết định sai** — attribution bị cắt khúc dẫn tới phân bổ ngân sách sai. Đây là chi
   phí lớn nhất và là chi phí **không ai đo**.
3. **Chi phí trải nghiệm** — người dùng cuối đi qua nhiều chặng chuyển tiếp không nhất quán, mỗi
   chặng một thương hiệu, mỗi chặng một cơ hội rơi.

### 2.3 Vì sao các công cụ rời không tự hợp nhất

Vì mô hình kinh doanh của họ **đòi họ không hợp nhất**. Một shortener bán số link và số click. Một
công cụ deep link bán số install attributed. Một nền tảng bio bán số view. Hợp nhất cả ba vào một
báo cáo duy nhất nghĩa là **ba nhà cung cấp cùng phải chấp nhận một định nghĩa chung về "một lượt
chạm"** — và không ai trong số họ có động cơ làm điều đó, vì định nghĩa của họ chính là sản phẩm
của họ.

Đây là một **vấn đề cấu trúc thị trường**, không phải vấn đề kỹ thuật. Và đó là lý do lời giải phải
là một nền tảng mở, sở hữu toàn bộ chuỗi, chứ không phải một integration layer.

---

## 3. Luận điểm: một Link, một lớp Event

### 3.1 Hai bất biến

Toàn bộ kiến trúc 9Link quy về hai câu, được ghi ở đầu `CLAUDE.md` và được đánh dấu **BẤT BIẾN —
không được vi phạm**:

> **① Một Link, chín năng lực** — cùng một URL bật/tắt bất kỳ trụ cột nào mà **KHÔNG đổi địa chỉ**.
>
> **② Một lớp dữ liệu duy nhất** — mọi `Event` (`click`/`scan`/`view`/`convert`/`earn`/`impression`)
> của **mọi** trụ đổ về **một** kho. Đây là *moat*, không phải feature. **Cấm** tạo data-store song
> song phá vỡ điều này.

Bất biến ① không phải khẩu hiệu tiếp thị — nó là một ràng buộc kỹ thuật có thể kiểm chứng được:

```
GET /{key}   ── page tắt, không routing, không cổng ──▶  302 Location: <đích>
GET /{key}   ── gắn bio page ─────────────────────────▶  200 HTML bio
GET /{key}   ── bật cổng quảng cáo ────────────────────▶  200 HTML chặng 1 → 302
GET /{key}   ── bật password ─────────────────────────▶  200 form → 302
GET /{key}   ── bật routing geo/device/AB ─────────────▶  302 Location: <đích THEO LUẬT>
```

**Cùng một địa chỉ.** Không route mới, không subdomain, không path prefix. Tắt bio page ⇒ **cùng
URL** quay lại 302 tức thì. Đây là nghĩa đen của "một link, chín năng lực", và nó có lý do thương
mại: một QR đã in, một chiến dịch đã chạy, một bio đã dán ở 40 nơi — **địa chỉ đó là tài sản của
khách**, không phải của chúng tôi. Đổi địa chỉ là phá tài sản đó.

### 3.2 Bất biến ② trả tiền như thế nào

Một bảng duy nhất:

```sql
CREATE TABLE events (
  event_id UUID, event_type LowCardinality(String), ts DateTime64(3),
  link_id String, workspace_id String, domain String, key String,
  country LowCardinality(String), device LowCardinality(String),
  os LowCardinality(String), browser LowCardinality(String),
  referrer String, ua_hash String,
  variant String,        -- A/B
  click_id String,       -- attribution click → convert
  event_name String,     -- goal của convert; KẾT CỤC của impression
  value Float64, currency LowCardinality(String),
  is_bot UInt8, chain_key String
) ENGINE = MergeTree PARTITION BY toYYYYMM(ts)
  ORDER BY (workspace_id, link_id, ts);
```

Sáu `event_type` — `click`, `scan`, `view`, `convert`, `earn`, `impression` — **cùng một bảng, cùng
một khoá sắp xếp**. Hệ quả:

| Câu hỏi kinh doanh | Câu truy vấn |
|---|---|
| Phễu trọn vòng | `funnel: view → click → convert → earn` — **một** `GROUP BY event_type` |
| Link nào ra tiền | `click_id` join `convert` — attribution query-time, **không** bảng phụ |
| Biến thể A/B nào thắng | `variant` trên cùng hàng với `convert` — không cần export sang một tool thí nghiệm |
| Chuỗi chạm đa kênh | `chain_key` — nối QR → web → app **mà không thu thập thêm định danh mới** |
| Quảng cáo có lãi không | `impression` (đếm lượt) đối chiếu `ad_earnings` (tính tiền) — hai tầng, một nguồn lượt |

**Không một câu nào trong số này khả thi nếu dữ liệu nằm ở năm kho của năm nhà cung cấp.**

### 3.3 Ba quy tắc giữ cho bất biến ② không mục nát

Một bất biến chỉ có giá trị nếu nó **không bị xói mòn bởi quyết định tiện lợi**. Ba quy tắc:

1. **Additive-only.** Tính năng mới thêm **cột**, không thêm **bảng**, và không thêm DDL khi chỉ
   cần một `event_type` mới (`LowCardinality` nhận giá trị mới miễn phí). `view` (M7), `convert`
   (M4), `earn` (M8), `impression` (M24) đều vào **cùng bảng** với **zero hoặc một** cột mới.
2. **Không PII thô.** Chỉ `country/device/os/browser/referrer` + `ua_hash`. `chain_key` là HMAC có
   salt, **xoay theo tuần** — nối được chuỗi, không suy ngược được con người.
3. **Tiền không nằm trong Event.** `impression.value = 0`. CPM đổi theo hợp đồng; ghi tiền vào một
   kho append-only là **tự khoá mình khỏi việc sửa sổ**. *Event đếm sự kiện, ledger tính tiền.*

Quy tắc 3 là ranh giới quan trọng nhất của cả mô hình kiếm tiền, và nó được triển khai bằng hai hệ
thống có **bảo đảm khác nhau**:

```
ClickHouse events       "có bao nhiêu lượt, lượt nào bị loại và vì sao"    append-only, at-least-once
        │  settle job (worker)
        ▼
Postgres ad_earnings    "phải trả bao nhiêu, trạng thái gì, ai thu hồi"     idempotent, khoá tự nhiên 5 cột
        │  payout claim (CAS)
        ▼
Postgres payout_requests "đang chi trả đợt nào"                              một yêu cầu mở mỗi workspace
        │  operator duyệt
        ▼
ngoài hệ thống          chuyển tiền thật (v1 có chủ ý dừng ở approved)
```

---

## 4. Sản phẩm: chín trụ trên cùng một URL

| # | Trụ | Động từ | Lõi | Lớp lộ trình |
|---|---|---|---|---|
| 1 | **Create** | `/create` | short/branded link, custom domain (verify DNS thật), bulk JSON/CSV, REST API | **Land** |
| 2 | **Enrich** | `/enrich` | QR động **có brand** (đổi đích không đổi QR), UTM builder, OG preview, deep link | **Land** |
| 3 | **Route** | `/route` | geo/device/ngôn ngữ/giờ, A/B split sticky, deferred deep link, **WhatsApp routing** | **Land** |
| 4 | **Publish** | `/publish` | link-in-bio **native trên chính Link**, block system, theme, RTL per-page | Expand |
| 5 | **Measure** | `/measure` | analytics real-time, conversion, attribution, campaign rollup, retargeting pixel | **Land** |
| 6 | **Optimize** | `/optimize` | A/B significance (z-test 2 tỷ lệ), declare-winner, funnel, gợi ý AI BYO | Platform |
| 7 | **Monetize** | `/monetize` | cổng quảng cáo · affiliate auto-tag · tip/checkout · ví · chi trả | Expand |
| 8 | **Govern** | `/govern` | password/expiry/click-limit, chống phishing, RBAC, SSO, audit, **data residency** | **Land** |
| 9 | **Connect** | `/connect` | REST API, OpenAPI 3.1, SDK, node n8n, webhook OUT ký HMAC | Expand |

### 4.1 Luật hai tầng tên

Tên trụ + route (`Create`/`/create`) là **canonical**, giữ nguyên trong code, API, OpenAPI tag, docs
kỹ thuật. Nhưng **UI hiển thị tên thân thiện theo locale** (nguồn `common.nav.*` trong
`messages/{vi,en,ar}.json` — ví dụ vi "Rút gọn link", en "Short links"). Đây là một quyết định có
lý do: người dùng không nghĩ theo động từ kiến trúc, họ nghĩ theo việc họ đang làm.

### 4.2 Ba quyết định sản phẩm đáng chú ý

**① Bio page là một thuộc tính của Link, không phải một sản phẩm riêng.** Không có `/@handle`,
không có subdomain, không có "chuyển sang Link Pages". Chỉ một cột `Link.pageId`
(`onDelete: SetNull`). Lý do: bất biến ①. Và lý do sâu hơn — một bio page ở một địa chỉ **khác** là
một tài sản thứ hai phải xây dựng thương hiệu, phải SEO, phải dán lại QR. Chúng tôi từ chối bán
cho khách một vấn đề mới.

**② "Declare winner" là NGƯỜI BẤM, không phải tự động.** Trụ 6 tính p-value **chỉ khi đủ mẫu**
(không số ảo), và việc áp kết quả là một hành động có chủ ý của con người, **đảo ngược được**, đi
qua đúng đường routing của trụ 3. Tương tự, output của LLM **không bao giờ** auto-apply. Đây là một
lập trường sản phẩm: trong một công cụ ra quyết định tiếp thị, một gợi ý sai được áp tự động đắt
hơn nhiều so với một gợi ý đúng phải bấm thêm một lần.

**③ Đơn giản mặc định, quyền năng khi cần.** Tạo link ≤ 3 giây; tám trụ còn lại ẩn sau, mở khi
cần. Onboarding có một việc: tạo link đầu tiên.

### 4.3 Đa ngôn ngữ và RTL là P0, không phải bản địa hoá về sau

Ba locale đầy đủ (`vi`, `en`, `ar`) với **1.683 khoá i18n**, RTL thật (đã verify bằng `curl` trang
thật: `dir="rtl"` + chữ Ả Rập), và **ba trang vô danh** — trang 402 trần lưu lượng, form mật khẩu
link, trang cổng quảng cáo — đều render ba ngôn ngữ từ `Accept-Language`.

Cơ chế: cookie `NEXT_LOCALE`, **không middleware**, **không URL-routing theo locale** (ADR 0014).
Đây là một ràng buộc kiến trúc chứ không phải sở thích: một middleware i18n sẽ chạy trên **mọi**
request, kể cả `/{key}`, và sẽ phá bất biến "redirect không đổi".

Vì sao Ả Rập là P0: đó là beachhead (§11). Một sản phẩm MENA mà RTL là bản vá về sau thì không phải
sản phẩm MENA.

---

## 5. Kiến trúc: bốn mặt phẳng và một đường nóng

```
┌─ EDGE / PHỤC VỤ ─────────────────────────────────────────────────────────┐
│  Nhận click → chạy guard (trụ 8) → chạy rule (trụ 3) → ghi Event ASYNC   │
│  → trả 302. Chống abuse/phishing tại đây.                                │
│  Hiện trạng: Next.js route handler + reverse proxy. Đích: Cloudflare PoP. │
├─ CORE API ───────────────────────────────────────────────────────────────┤
│  Next.js 15 App Router + Prisma. 9 trụ = 9 module dùng CHUNG Link model.  │
│  /ee tách riêng qua 5 ổ cắm — KHÔNG import ngược, gác bằng cổng máy.      │
├─ DATA (UNIFIED) ─────────────────────────────────────────────────────────┤
│  Postgres : Link, Page, rule, workspace, user, SỔ TIỀN  (1..N cụm)        │
│  ClickHouse: Event append-only — MỘT BẢNG                (1..N cụm)       │
│  Redis    : cache + rate-limit + 2 stream (events, webhooks)              │
├─ APP/UI ─────────────────────────────────────────────────────────────────┤
│  Dashboard "tạo 1 link" → mở dần trụ. i18n + RTL = P0.                    │
└──────────────────────────────────────────────────────────────────────────┘
```

Toàn bộ stack là **100% OSS, không khoá nhà cung cấp**: Next.js · Prisma · Postgres · ClickHouse ·
Redis · Better Auth · GrowthBook (tuỳ chọn) · n8n (tuỳ chọn) · Stripe · `qr-code-styling`. Không một
license độc quyền nào trong dependency manifest — và manifest bị **đóng băng bằng hash**
(`.gate/frozen.sha256`, `scripts/gate-deps.mjs`).

### 5.1 Quyết định kiến trúc quan trọng nhất

> `Event` nằm ở **ClickHouse**, không phải Postgres. Append-only. Tách khỏi redirect bằng **hàng đợi
> Redis**. Đừng để việc ghi analytics làm chậm cú 302.

Triển khai cụ thể: `buildClickEvent()` chạy **đồng bộ** (thuần CPU, ~15 phép so chuỗi), còn
`enqueueEvent()` nằm trong `after()` của Next.js — **không chặn response**. Worker rút stream và
`INSERT` lô vào ClickHouse. Người xem nhận 302 trước khi Event chạm Redis.

### 5.2 "Zero-cost" — lời hứa có thể đo

Mọi tính năng mới trên đường phục vụ phải chứng minh **hai vế**:

| Vế | Nội dung | Ví dụ cổng |
|---|---|---|
| **Âm** | Tính năng **tắt** ⇒ phản hồi **byte-identical** mốc trước, **không thêm một lượt I/O** | `GATE-ADGATE` "zero-cost link thường" |
| **Dương** | Tính năng **bật** ⇒ hành vi mới thật sự xảy ra | `GATE-ADGATE` "đối chứng dương" |

Chỉ có vế âm là một cổng **xanh vì nhánh chưa bao giờ chạy**. Đây là bài học M33 được tổng quát thành
luật: *mọi cổng phải tự chứng minh nó ĐỎ được*.

Cơ chế cho phép giữ lời hứa đó qua 6 tính năng: `ResolvedLink` là một **`select` tường minh**, mỗi
trường được kéo vào hot path bởi **một ADR riêng**:

| Trường | Trụ | ADR |
|---|---|---|
| `routing` | 3 Route | **0006** — ngoại lệ hẹp đầu tiên với ADR 0003 |
| `trackConversion` | 5 Measure | **0008** |
| `pageId` | 4 Publish | **0016** |
| `quarantined` | 8 Govern | M18/T18.3 |
| `serveBlocked` | 8/billing | **0041** |
| `adGate` | 7 Monetize | **0043** — ngoại lệ kéo cả **config** (3 khoá vô hướng, cap 300 byte), vì edge phải canh tuổi token |

Mọi blob còn lại — `utm`, `qrConfig`, `og`, `deepLink`, `goals`, `pixels`, `blocks`, `theme` —
**không bao giờ** vào hot path.

### 5.3 Ba tiến trình

| Tiến trình | Vai trò | Đặc điểm |
|---|---|---|
| **web** | Next.js standalone | Bốn nhóm route: đường phục vụ · công khai · auth · dashboard/API. Không giữ trạng thái |
| **worker** | Ba loop song song | ① ingest Event ② delivery webhook ③ job nền (**kể cả kết sổ tiền**). Cùng image với web |
| **ops CLI** | Ứng cứu sự cố | `blocklist` · `ops:ads` · `ops:margin` · `ops:payout` · `rotate:auth-secret`. **Không cần deploy** |

Chi tiết đầy đủ: [`docs/c4/02-container.md`](c4/02-container.md) và
[`docs/c4/03-component.md`](c4/03-component.md).

### 5.4 Những gì cố ý không có

| Không có | Vì sao |
|---|---|
| Message broker riêng (Kafka/SQS) | Redis Stream đủ cho hai hàng đợi; thêm broker là thêm một nơi Event có thể mất |
| Object storage (S3/R2) | **Không có upload nào**: block bio cap 30/16KB, cấm custom HTML, QR logo là data-URL. Bỏ storage là bỏ cả một lớp lỗ hổng |
| Service analytics riêng | Tách ra sẽ tạo nơi thứ hai định nghĩa "trial/success" — đúng thứ ADR 0024 mua bằng hợp đồng SQL có test |
| CDN cache cho redirect | `no-store` trên mọi phản hồi. Một proxy giữ 302 ⇒ cú bấm thứ hai không đếm được |
| Scheduler/cron riêng | Job nền trong worker, **lịch ở Postgres** — vì `FLUSHDB`/eviction làm việc quét ngừng **im lặng** |

---

## 6. Mô hình kinh doanh: pivot 2026-08-25

### 6.1 Trước và sau

| | Trước M36 | Sau M36 |
|---|---|---|
| Cloud bán gì | Ba bậc Free/Pro/Business theo **hạn mức** (link · domain · page · lưu lượng) | **Không bán dung lượng.** Mọi hạn mức là `UNLIMITED` ở **mọi** gói |
| Nguồn thu chính | SaaS subscription | **Quảng cáo trên cổng của link** (9Link **là** mạng) + take-rate GMV affiliate |
| Nguồn thu phụ | — | Giấy phép `/ee` doanh nghiệp: white-label · SSO/SAML · data residency · audit/SLA |
| Mặc định sản phẩm | Link thường | **Cổng kiếm tiền BẬT MẶC ĐỊNH cho link MỚI**, kèm công tắc tắt rõ ràng, **không hồi tố** link cũ |

Trong mã, pivot này là **một hằng số**: mọi `QuotaFeature` (`links` · `domains` · `pages`) và trần
lưu lượng tháng đều `UNLIMITED`. Và nó có một lời cảnh báo được viết sẵn cho người đến sau:

> *"Đó không phải 'ngưỡng tạm nới rộng' — nó là **mô hình kinh doanh**. Đặt lại một con số hữu hạn
> vào đây là bán thứ đã tuyên bố cho không, và người phát hiện sẽ là khách chứ không phải cổng."*
> — `apps/web/lib/billing/plans.ts`

Ba gói `free`/`pro`/`business` **vẫn tồn tại trong mã**, có chủ ý: `plan` là cột do **webhook
Stripe** ghi. Rút một id ra khỏi danh sách sẽ biến mọi hàng mang id đó thành "plan lạ" ⇒ fallback
`free` **im lặng**, đúng vào ngày ai đó còn subscription đang chạy. Thứ đổi là **ý nghĩa**: gói
không còn giới hạn khả năng, nó chỉ còn đánh dấu một quan hệ thương mại.

### 6.2 Bốn đường kiếm tiền quanh một link

| # | Đường | Trạng thái | Cơ chế |
|---|---|---|---|
| ① | **Cổng quảng cáo trên link rút gọn** | ✅ mã đầy đủ, inert trên prod | Cổng 2 chặng + đếm ngược server-side + captcha tự chủ → `impression` → `ad_earnings` |
| ② | **Ô quảng cáo trên bio page** | ✅ (M38) | `page-slot.ts` đồng bộ, zero I/O, kho nằm trong RAM |
| ③ | **Referral** | ✅ (M34/M38) | `/r/<code>` → `/signup?ref=`; hoa hồng `referralBps` trên doanh thu tuyến dưới, ghi thành `kind="referral"` trong **cùng** sổ |
| ④ | **Take-rate GMV affiliate** | ✅ `/ee` | `floor(amount × bps / 10000)` thu bằng `application_fee_amount` trên destination charge |
| — | **Trả phí để BỎ cổng** | ⚪ chưa dựng | Được chốt trong phạm vi pivot, chưa có mã |
| — | **Giấy phép `/ee`** | ✅ cơ chế, chưa có khách | SSO/SAML (193 dòng) · residency policy (202 dòng + 304 dòng test) · audit/SLA |
| — | ↳ trong đó **white-label** | ⚪ **chưa có mã** | 0 dòng trong `ee/`, 0 ổ cắm trong 5 ổ — xem §12.6. Bản 1.0 gộp nó vào dấu ✅ của hàng trên; đó là một câu rộng hơn mã |

### 6.3 Vì sao mô hình này thẳng hàng lợi ích

Trong một SaaS bán dung lượng, nền tảng có động cơ **đặt trần thấp** và khách có động cơ **lách
trần**. Trong mô hình này:

```
lưu lượng THẬT của publisher  →  lượt hiển thị  →  doanh thu mạng  →  phần chia cho publisher
        ▲                                                              │
        └──────────────── publisher có động cơ ở lại ◀─────────────────┘
```

Cả hai bên cùng muốn **một con số tăng**: lượt xem thật của người thật. Đây là lý do toàn bộ phần
chống gian lận (§7.3) được thiết kế theo hướng *bảo vệ chất lượng lưu lượng*, chứ không phải
*hạn chế lưu lượng*.

### 6.4 Cái van duy nhất còn lại — và vì sao nó phải tồn tại

"Miễn phí không giới hạn" ≠ "không có van". `workspaces.serveAllowanceOverride` là trần lưu lượng do
**operator** đặt cho một workspace lạm dụng. Nó ra đời **cùng ngày** với việc gỡ trần gói, và đó
không phải trùng hợp:

> Gỡ trần mà không có van thay thế nghĩa là **không còn gì chặn** kẻ bơm lưu lượng vào link **không
> bật cổng** — chi phí phục vụ thuần, **0 đồng** quảng cáo bù. Và trước cột này, thứ duy nhất làm
> được là xoá tài khoản: một phản ứng vừa quá tay vừa không dừng được kẻ tạo tài khoản mới.
> — `DECISIONS.md`, 2026-08-25

Kèm theo: nó là đường **duy nhất** còn kích hoạt được toàn bộ cơ chế trần M22 (job quét · cảnh báo
80/100 · ân hạn 48h · trang 402 trung tính ba ngôn ngữ). `0` = **CHẶN**, `null` = **KHÔNG GIỚI HẠN**
— hai giá trị này **không được gộp bằng một phép kiểm falsy**.

---

## 7. Kinh tế học của cổng quảng cáo

Đây là phần quan trọng nhất của mô hình kinh doanh và cũng là phần có nhiều ràng buộc kỹ thuật
nhất, vì nó là chỗ **tiền thật chảy ra khỏi túi 9Link**.

### 7.1 Trải nghiệm người xem

```
GET /{key}
   │  (sau password, TRƯỚC click-limit, KHÔNG chạy khi link gắn bio page)
   ▼
200 — trang chặng 1: ô quảng cáo + đếm ngược + [Tiếp tục] (LUÔN CÓ TRONG DOM)
   │  POST gate_token + gate_answer
   ▼
200 — trang chặng 2 (nếu cấu hình 2 chặng)
   │  POST
   ▼
302 về chính path · Set-Cookie: 9lg_<linkId> (vé 60 giây)
   │
   ▼
GET /{key} — có vé ⇒ 302 tới ĐÍCH. Đuôi 302 chuẩn KHÔNG ĐỔI MỘT DÒNG.
```

Bốn quyết định thiết kế, mỗi cái một lý do:

| Quyết định | Lý do |
|---|---|
| **Vé đã-qua-cổng**, không để POST tự trả 302 tới đích | Để POST tự 302, nó phải chạy lại routing + A/B + `9l_cid` + Event click — tức hoặc **chép**, hoặc **bóc đuôi 302 ra hàm dùng chung trong file đóng băng**. Với vé, đuôi ấy không đổi một dòng |
| **Đếm ngược do SERVER canh**, bằng tuổi của token đã ký | Chuỗi ký phủ bốn thứ, mỗi thứ đóng một cách bịa ra tiền: `9la:1:<linkId>:<stage>:<iat>:<nonce>` — linkId (token link rẻ không farm sang link đắt) · stage (token chặng 1 ≠ chặng 2) · iat (**đếm ngược không tua được**) · nonce (**một-lần**) |
| Nút "Tiếp tục" **luôn có trong DOM**; JS chỉ vô hiệu hoá tới hết giờ | Tắt JS vẫn dùng được **và vẫn không rút ngắn được một giây nào**. Một cổng chỉ hoạt động khi có JS là một cổng mà người thật trên trình duyệt chặn-JS bị kẹt |
| **Không có lối ra 4xx/5xx nào cho người xem** | Token hỏng, captcha sai, bấm sớm, hết hạn — tất cả đều trả một trang dùng được. Người đứng ở đây chỉ muốn tới cái link; họ không phải kẻ tấn công, và ngay cả khi là kẻ tấn công thì trả lỗi cũng không ngăn được gì |

**Captcha tự viết, và tài liệu nói thẳng nó yếu.** Không hCaptcha/reCAPTCHA/Turnstile, vì ba lý do:
chúng gửi dấu vết của **mọi visitor của khách** cho một bên thứ ba mà khách **không chọn** (xung
đột trực tiếp với trụ 8 và residency); chúng là **script chặn đường** trên đúng trang đang bán tốc
độ; và chúng là **dependency ToS độc quyền** — đúng loại đã buộc chúng tôi gỡ app Zapier chính
thức. Thử thách được suy **từ nonce** bằng HMAC ⇒ server tái tạo đáp án **mà không lưu một dòng
trạng thái nào**.

Và đây là chỗ phải trung thực: **phòng tuyến thật không phải captcha**. Theo thứ tự sức nặng:
**giữ tiền 30 ngày** → dedup/velocity/bot-UA → captcha.

### 7.2 Vị trí của cổng trong chuỗi guard — ba lý do, không phải tiện tay

```
resolveLink → isLinkRedirectable → password → CỔNG QUẢNG CÁO → click-limit → bio page → routing → 302
                                                  ▲
                              (a) SAU password  (b) TRƯỚC click-limit  (c) KHÔNG chạy khi có bio page
```

- **(a) Sau password** — không lộ **gì** trước mật khẩu, kể cả sự tồn tại của quảng cáo.
- **(b) Trước click-limit** — một người xem đi qua cổng phát sinh **hai** lượt GET. Đặt sau
  click-limit thì hạn mức mà chủ link đặt cho **người** tự hụt **một nửa**, im lặng. Ở vị trí này,
  lượt trả trang cổng **không đếm**, và chỉ lượt **phục vụ thật** (có vé) mới tiêu một lượt.
- **(c) Bio page thắng** — bio là **đích cuối**, không phải chặng chuyển tiếp; chèn quảng cáo trước
  nó là **một sản phẩm khác**. Và khi page tắt, link về hành vi 302 thường chứ không rơi vào cổng.

### 7.3 Chống gian lận: hai hệ quả tách rời

Đây là nguyên tắc **đứng trên tất cả** của nhánh kiếm tiền (ADR 0043 §4):

| | |
|---|---|
| **① Người xem có được phục vụ không** | **LUÔN LUÔN CÓ**, không ngoại lệ |
| **② Chủ link có được tính tiền không** | chỉ khi **mọi** vế kiểm được **và** sạch |

> Gộp lại là cách mọi nền tảng paid-to-click tự bắn vào chân mình: một dương-tính-giả biến thành
> "link chết" với người thật — mà người thật **không khiếu nại**, họ chỉ không quay lại.

Hệ quả thứ hai, ít hiển nhiên hơn và đắt không kém: **"không kiểm được" ≠ "sạch"**. Redis chết,
thiếu khoá HMAC, cấu hình lệch ⇒ `unverifiable` ⇒ **không tính tiền**, còn visitor **vẫn đi tiếp**.

Và **ghi cả lượt bị loại**:

```
event_type = 'impression'
event_name = counted | bot | replay | duplicate | velocity | unverifiable
value      = 0
```

Chỉ ghi lượt sạch thì phần bị loại thành một **hố im lặng** — và cái hố đó chính là chỗ gian lận
sống. Đây cũng là lý do một marker bot **sai** đắt hơn lọt vài bot: substring `bot` từng khớp
**CUBOT** (một hãng điện thoại Android có thật, phổ biến ở đúng thị trường MENA của wedge) ⇒ vì
`CLICK_DEDUP` lọc `is_bot = 0` và overview join nhánh click, **đơn hàng của khách thật biến mất im
lặng khỏi báo cáo**.

### 7.4 Sổ cái: đơn vị, khoá, và trạng thái

| Quy tắc | Triển khai |
|---|---|
| **Tiền là minor units, số nguyên, `floor` ở mức minor** | `amountMinor = floor(impressions × cpmMinor / 1000)`. `Int` cho một **mức giá**, `BigInt` cho một **tích luỹ**. `floor` nghiêng về phía creator |
| **Khoá tự nhiên 5 cột** | `(workspaceId, day, country, kind, sourceWorkspaceId)` — **toàn bộ** vế idempotent: kết sổ chạy lại **không nhân đôi tiền** |
| `sourceWorkspaceId` **NOT NULL** | Postgres coi `NULL ≠ NULL` ⇒ UNIQUE có cột nullable **không chặn được trùng** ⇒ job chạy lại nhân đôi tiền, im lặng |
| **CPM GHIM tại thời điểm kết sổ** | Không tra lại. Một bảng giá mới không được viết lại lịch sử |
| **Trạng thái** | `pending` → `available` (sau `holdDays`) → `claimed` (CAS) → `paid`; nhánh `reversed` có `reversedSource` |
| **Quy tiền về link bằng PHÂN RÃ** | Largest-remainder trên số lượt được tính — **không** thêm cột `linkId` vào bảng tiền |

Hai quyết định cuối đáng giải thích vì cả hai là kết quả của việc **làm thật rồi đo cái giá**:

**Vì sao không thêm `linkId` vào bảng tiền?** Vì số dòng sổ mỗi ngày sẽ nhân lên bằng
**số link có lưu lượng × số quốc gia** thay vì chỉ số quốc gia. Một publisher 100 link chạm 10
nước là **1.000 dòng/ngày cho một người**; ở quy mô 10k publisher là **hàng triệu dòng/ngày trên
một bảng tiền** — thứ phải khoá, sao lưu, đối soát từng dòng, và phải `ALTER` khi lược đồ đổi.

**Vì sao phép chia không phải một ước lượng?** Vì trong một nhóm `(workspace, ngày, quốc gia)`,
**mọi lượt có cùng một CPM** — đó chính là *định nghĩa* của nhóm mà `computeEarningLines` dựng ra.
Nên tiền **tỷ lệ chính xác** với số lượt; thứ duy nhất không chia hết là phần dư dưới một đơn vị
minor, và largest-remainder **phát** phần dư đó thay vì đánh rơi. Bất biến thu được mạnh hơn một
cột: `tổng các phần === số tiền của nhóm, luôn luôn`.

### 7.5 Bảng giá — chỗ duy nhất một con số giá đi vào hệ

```
AdRates {
  currency            : ISO lowercase — MỘT đồng tiền cho cả bảng (trộn nhiều đồng là bài toán FX)
  cpmMinorByCountry   : Record<ISO-2, minor units / 1000 lượt>
  defaultCpmMinor     : cho nước không có trong bảng — 0 = không trả cho lưu lượng đó
  referralBps         : hoa hồng giới thiệu (2000 = 20%) — 0 = không có chương trình
  holdDays            : giữ tiền bao nhiêu ngày trước khi cho rút
  payoutThresholdMinor: ngưỡng rút tối thiểu
}
```

Bảng giá **không nằm trong Core**. Core biết **cách** tiền sinh ra / giữ lại / mở khoá; nó cố ý
**không biết** CPM nước nào bao nhiêu. Nhờ ranh giới đó, **job kết sổ chạy trong worker của Core**
mà Core không bao giờ import `/ee`.

| Nguồn | Ai dùng |
|---|---|
| `AdRateCard` (Postgres), sửa từ `/govern/rates` | 9Link Cloud — operator cấp instance |
| `AD_NETWORK_9LINK_CPM` (env) | Instance `/ee` cấu hình tĩnh |
| **Không có** (`rates() == null`) | **Bản OSS/BYO** — tiền về thẳng từ mạng của chính chủ instance, 9Link không đứng giữa ⇒ **không có gì để ghi sổ** |

**Bảng giá tham chiếu đang chờ kích hoạt** (đối chiếu từ ouo.io, do người sáng lập duyệt):

| | CPM (minor units / 1000 lượt) |
|---|---|
| Mặc định | **150** |
| US | **410** |
| AU | **500** |
| Ngưỡng rút | **500** |

⚠️ **Trạng thái production hôm nay: `ad_rate_cards` = 0 hàng, `ad_earnings` = 0 dòng.** Cơ chế đã
lên nhưng **chưa từng chạy thật một lần nào**. Cảnh báo trên chính màn hình `/govern/rates`:
*"cấp CPM = NHẬN NGHĨA VỤ TRẢ TIỀN"*. Đó là trạng thái **đúng thiết kế**, không phải nợ — và nó là
lý do §12 tồn tại.

### 7.6 Đối soát biên — hệ thống được phép nói "tôi không biết"

Một lỗ đắt hơn mọi lỗ khác trong nhánh kiếm tiền:

> `cpmMinorByCountry` là số ta **TỰ KHAI**, và `ad_earnings` là **NGHĨA VỤ TRẢ** tính từ chính con
> số đó. Không có doanh thu thật của mạng thì hệ thống hoàn toàn không biết mạng **thực sự** trả
> bao nhiêu — nên nếu eCPM thật thấp hơn CPM ta khai, **ta lỗ trên mọi lượt, im lặng, và mọi bất
> biến kế toán vẫn cân** (sổ nợ đúng, tiền trả đúng, chỉ là trả nhiều hơn thu).

Vì vậy có `ad_network_revenue`: doanh thu **thật** do mạng báo về, theo `(ngày, quốc gia, đồng
tiền, nguồn)`, **nhập từ ngoài**, có khoá tự nhiên và hai CHECK về chữ hoa/thường. Nó **không bao
giờ** được dùng làm nguồn để **tính** tiền phải trả — nó chỉ để **đối chiếu**.

Và đây là quyết định thiết kế quan trọng nhất của phần này: **khi thiếu dữ liệu, hệ từ chối kết
luận.** Mọi ngày không đủ cả hai vế được báo `insufficient` — **không** suy ra lãi, **không** suy
ra lỗ. Một con số biên bịa ra theo chiều "lãi" sẽ tắt đúng cái chuông cần reo.

Hai chi tiết kế toán đáng ghi lại vì chúng là loại lỗi không ai thấy:

- `day` là `@db.Date`, **cùng mốc và cùng kiểu** với `ad_earnings.day`. Một hàng mang thêm phần giờ
  sẽ **vô hình** với truy vấn đối soát.
- Khoá tự nhiên có `source` từ M37, vì từ đó có **hai** nguồn doanh thu (nhập tay từ mạng đối tác,
  và 9Link Ads tự sinh). Khoá cũ làm hai nguồn **ghi đè nhau** ⇒ biên của ngày đó **nói dối theo
  chiều tắt chuông**.

### 7.7 Chi trả — và ranh giới của v1

```
publisher: ví (available · pending · reversed · paid, theo link và theo nước)
   → hồ sơ nhận tiền (mã hoá, PAYOUT_PROFILE_KEY) → step-up 2FA (phiên ≤ 900s)
   → requestPayout: MỘT yêu cầu mở mỗi workspace (partial unique index)
   → CAS: ad_earnings available → claimed (ĐẾM ROWCOUNT, không đọc số dư)
   → operator duyệt tại /govern/payouts hoặc pnpm ops:payout
   → paid.  Từ chối ⇒ releasePayoutClaim (WHERE status='claimed' ⇒ dòng đã paid không bị kéo ngược)
```

**Chuyển tiền thật diễn ra NGOÀI hệ thống, có chủ ý.** v1 dừng ở `approved`/`paid`. KYC, thuế
(W-9/W-8BEN, 1099), merchant-of-record là việc `[human]`, không phải nợ mã. Chúng tôi chọn nói rõ
điều này thay vì để một đối tác phát hiện ra trong một cuộc đối soát.

Hai ràng buộc bảo mật ở cửa tiền ra:

- **Step-up 900 giây là một con số về bảo mật, không phải về tiện lợi.** Nới nó là nới đúng cửa sổ
  mà một cookie bị chiếm dùng được ở **bề mặt duy nhất trong repo chuyển tiền ra khỏi hệ**.
- **Chữ trên màn hình tiền phải được tính từ cấu hình**, không hardcode. Chuỗi `errStepUp` từng ghim
  chữ "15 phút" trong ba locale trong khi trần là một biến môi trường — instance đặt
  `PAYOUT_STEPUP_MAX_AGE_SEC=300` sẽ nói dối người dùng ở đúng màn tiền ra. Làm tròn **lên**: chữ
  ở màn tiền được phép rộng hơn sự thật, không được hẹp hơn.

---

## 8. Chủ quyền dữ liệu: wedge MENA

### 8.1 Ba mức — và mỗi mức là một lời hứa khác nhau

Đây là chỗ tài liệu marketing thường nói quá. Chúng tôi nói đúng ba mức, và nói rõ mức nào đang ở
đâu:

| Mức | Lời hứa | Cơ chế | Trạng thái |
|---|---|---|---|
| **1 — Nhãn** | Mỗi workspace có một vùng (`US` \| `UAE` \| `EU`) | `workspaces.region` | ✅ từ M5 |
| **2 — Event theo cụm** | **Event** của workspace nằm ở cụm ClickHouse của vùng đó | `WorkspaceRegionSpan` (bản đồ **thời gian → cụm**) · ghi `XADD events:stream:<clusterId>` · đọc fan-out theo span rồi gộp | ✅ từ M19. **Bản OSS/self-host luôn MỘT cụm** |
| **3 — PII theo cụm** | **Tài khoản, phiên, Link, Page** nằm ở cụm Postgres của vùng | Một schema → N cụm (`POSTGRES_CLUSTERS`) · `APP_REGION` · `region_directory` toàn cục | ✅ cơ chế từ M30 |

**Ba điều phải nói kèm, vì thiếu chúng là nói rộng hơn mã:**

1. **Prod 9Link hôm nay vẫn MỘT cụm** — chưa có endpoint UAE/EU.
2. **Bản OSS/self-host LUÔN một cụm**, mọi vùng → `"default"`. Không có env nào cứu được việc chỉ
   có một cụm, và tài liệu self-host **chỉ được nói mức 1**.
3. **Đổi vùng ⇒ dữ liệu cũ Ở LẠI cụm cũ.** Không có di trú.

### 8.2 Ranh giới open-core của residency

| | Core (AGPLv3) | `/ee` (commercial) |
|---|---|---|
| Sở hữu | **Seam + hạ tầng**: pool ClickHouse, `INSERT`, retry, `XACK`, fan-out đọc, `WorkspaceRegionSpan` | **Chính sách**: bản đồ `region → clusterId` từ `EE_RESIDENCY_CLUSTERS` |
| Không sở hữu | — | **Không cầm client ClickHouse riêng** |

Lý do `/ee` **chỉ trả bản đồ, không cầm client**: `/ee` tự mở client của nó ⇒ thành **hai đường ghi
Event** ⇒ phá bất biến "một lớp dữ liệu duy nhất" và làm poison-isolation chỉ còn che một nửa. Ổ
cắm trả đúng **một** thứ: `clusterId`.

### 8.3 Fail-closed là luật, không phải tuỳ chọn

> Chế độ hỏng chết người của residency **không phải** "hệ đứng". Nó là **"hệ vẫn chạy, mọi cổng vẫn
> xanh, dashboard vẫn ra số — trong khi dữ liệu vùng UAE đã chảy sang cụm US"**.

Vì vậy: không phân giải được cụm ⇒ **từ chối ghi/đọc**, event nằm chờ ở `events:stream:unrouted`
(có `MAXLEN`), **không bao giờ** rơi về cụm mặc định. Bốn luật fail-closed ở mức 3:

1. Cụm `default` **không được khai** trong JSON (khai vào ⇒ ném) — nó luôn đến từ `DATABASE_URL`.
2. Khai ≥1 cụm Postgres mà **không** có `EE_RESIDENCY_CLUSTERS` ⇒ **từ chối boot**.
3. **Tập** cụm Postgres khác **tập** cụm ClickHouse ⇒ **từ chối boot** (một vùng bị chẻ đôi: PII một
   cụm, Event một cụm khác). So bằng **tập**, không bằng số lượng — `uae` vs `uae_1` cùng kích
   thước mà lệch tên là đúng ca đã đốt một lần.
4. Truy vấn Postgres chạy **ngoài** mọi ngữ cảnh vùng ⇒ **ném**. Job nền phải nói ra cụm bằng
   `runInCluster(id, "lý do", fn)` — một lời khai **nhìn thấy được**, không phải một mặc định.

### 8.4 Nhãn residency sống trong schema, không trong một file doc rời

Mọi model trong `schema.prisma` mang:

```prisma
// @residency theo-vùng | toàn-cục | cần-quyết
// @pii name · slug · stripeCustomerId      (hoặc  @pii —  = đã duyệt và kết luận KHÔNG CÓ)
model Workspace { … }
```

Lý do không đặt bảng phân loại ở `docs/`:

> Một bảng phân loại đặt ở `docs/` sẽ **lệch với schema kể từ model tiếp theo** ai đó thêm vào — đó
> là cả một họ lỗi của repo này ("doc nói rộng hơn code"), và ở đây cái giá của nó là **PII của vùng
> UAE nằm trên đất Mỹ vì không ai nhớ dán nhãn cho bảng mới**.

Cổng `residency-schema-labels.test.ts` đọc **chính file schema** và ĐỎ khi có model thiếu nhãn.
`cần-quyết` chỉ hợp lệ **giữa chừng** một task — cổng ĐỎ nếu còn dù một bảng mang nhãn đó.

### 8.5 `region_directory` — tối thiểu hoá đến mức cực đoan

Ở mức 3, `users` sống **trong** vùng. Nhưng lúc một người gõ email vào ô đăng nhập, hệ **chưa biết**
vùng nào. Giải pháp là một bảng toàn cục, và nó được thiết kế để **không thể trở thành một danh
sách email**:

| Thuộc tính | Giá trị |
|---|---|
| Khoá | `HMAC-SHA256(email chuẩn hoá, DIRECTORY_PEPPER)` → 32 hex. **Không** email, **không** hash trần — không gian email là liệt kê được, `sha256(email)` là một định danh **giả vờ** ẩn danh |
| Pepper | **Không có fallback**, cố ý. Rơi về `BETTER_AUTH_SECRET` (thứ đã nằm ở **mọi** cụm) sẽ làm lời hứa "pepper không nằm cùng chỗ với bảng" thành **sai** — cầm được một cụm là đọc được cả danh bạ |
| Số cột | **Ba**. Không tên, không workspace, không thời điểm đăng nhập, không đếm lượt. Mỗi cột thêm vào là một mảnh PII được **nhân bản ra ngoài mọi vùng** |

### 8.6 Vì sao đây là wedge phòng thủ được

Ba thứ phải **cùng có mặt** để bán enterprise/government vùng Vịnh, và cả ba đều là P0 của repo
chứ không phải bản vá về sau:

| | |
|---|---|
| **RTL/Arabic hạng nhất** | 1.683 khoá × 3 locale, RTL verify bằng trang thật, **ba trang vô danh** cũng 3 ngôn ngữ |
| **WhatsApp routing** | `composeWhatsApp` → `wa.me/<E164>` như một preset của trụ 3 |
| **Data residency UAE** | Mức 2 + mức 3, fail-closed, nhãn PII trong schema, cổng máy đo **cả vế âm** (cụm này **không** thấy hàng của cụm kia) |

Thêm một thứ mà đối thủ đóng không có: **mã nguồn mở + self-host được**. Một cơ quan chính phủ có
thể đọc toàn bộ đường đi của dữ liệu của mình, tự vận hành, và không phải tin một lời hứa trong
hợp đồng.

---

## 9. Niềm tin: chống lạm dụng là rủi ro tồn vong

> §0.4 của `CLAUDE.md`: *"An toàn từ hạ tầng — chống phishing/abuse ở lớp Edge, ngày-1. **Domain bị
> blacklist = chết cả nền tảng.**"*

Đây không phải một mục tuân thủ. Với một dịch vụ rút gọn link, **domain dùng chung là tài sản lớn
nhất và mong manh nhất**: một đợt phishing đủ lớn khiến `9link.com` vào danh sách chặn của các
trình duyệt và nhà mạng thì **mọi link của mọi khách** chết cùng lúc, và không có bản vá nào.

### 9.1 Bốn lớp phòng thủ

| Lớp | Đứng ở đâu | Cơ chế |
|---|---|---|
| **① Lúc ghi** | write path | Quét **mọi** đích: `link.url` + mọi `dest` trong routing + mọi URL trong block bio. Blocklist nội bộ + Google Safe Browsing (tuỳ chọn) + feed threat-intel công khai. Trúng ⇒ `flagged` ⇒ edge **refuse** |
| **② Quét lại theo lịch** | worker loop ③ | **Lịch ở Postgres** (`nextScanAt`), Redis chỉ là bộ đệm. Link **đang kiếm tiền** ⇒ chu kỳ **dày gấp 4** (24h → 6h) |
| **③ Quarantine dính** | Postgres | `quarantinedAt` là **QUYẾT ĐỊNH**, tách khỏi `flagged` là **GIÁ TRỊ DẪN XUẤT**. Phán quyết ghi vào **không gian ĐÍCH** ⇒ xoá-rồi-tạo-lại **không rửa được** |
| **④ Van vận hành** | Redis + Postgres | Rate-limit theo IP/account/workspace · trần tạo link cho workspace mới · velocity · cầu dao instance (**chỉ ĐO**, chưa chặn) |

### 9.2 Ba lỗ hổng thật đã vá — nói thẳng

Chúng tôi ghi lại những lỗ này trong README công khai, vì một nhà cung cấp không kể ra lỗ của mình
thì không có cách nào kiểm chứng phần còn lại:

| Lỗ | Hậu quả nếu chưa vá |
|---|---|
| Một **dấu chấm cuối** né được cả blocklist (`evil.example.`) | Một đích phishing bị chặn vẫn phục vụ được bằng một biến thể chính tả |
| **Xoá-rồi-tạo-lại** rửa sạch phán quyết của operator | Kẻ bị chặn chỉ cần tạo lại link để về trạng thái sạch |
| **IdP của một tenant mint được phiên operator cấp instance** | Một khách hàng tự nâng mình lên quyền **toàn nền tảng** |
| Một entry `ABUSE_OPERATOR_EMAILS` **mồ côi** là lỗ hòm thư | Ai kiểm soát địa chỉ đó (alias cũ, domain catch-all) đăng ký + verify là thành operator. Prod nay **từ chối phục vụ** |
| `/report` vô danh: cap toàn cục tắt được kênh báo cáo của **cả instance** bằng 0,06 req/s | Một kẻ tấn công làm mù kênh báo cáo lạm dụng |
| POST chéo-site bơm hàng chờ bằng **IP thật của khách người khác** | Vừa DoS hàng chờ operator, vừa đổ lỗi IP cho nạn nhân |

### 9.3 Hai nguyên tắc thiết kế đáng chú ý

**① `degraded` = "KHÔNG BIẾT" ≠ "sạch".** Trước M18, GSB lỗi + một lượt `PATCH` đúng lúc là **rửa
sạch** cờ `flagged`. Nay ba trạng thái tách rời: `safe` / `degraded` / `hardReject`, kèm `source`
(**ai** kết luận). `degraded` ⇒ **không gỡ cờ cũ**, xếp lại lịch quét, không kết luận.

**② Báo cáo KHÔNG BAO GIỜ tự chặn link nào.** `/report` là công khai, vô danh. Nếu một báo cáo tự
chặn được link thì **bất kỳ ai cũng hạ được link bất kỳ với chi phí 0**. Báo cáo chỉ tạo
`AbuseCase`; phán quyết là của operator — một con người, có audit, có allowlist.

Và `quarantineSource` là **MÃ ENUM, không bao giờ là văn bản tự do**: DTO dùng **denylist**
(`Omit<T, "passwordHash">`) nên mọi cột mới **tự động** ra `GET /api/links`. Nội dung người báo cáo
/ chi tiết GSB / id case không được đi qua cột đó.

### 9.4 Tự lành — vì một dương-tính-giả không được thành án chung thân

Job **tự lành** theo luật hẹp: chỉ `source="scan"` + **N** lần quét sạch liên tiếp
(`SAFETY_CLEAN_STREAK`) + **chưa từng** bị operator "upheld". Thiếu vế tự lành thì một
dương-tính-giả = **link chết im lặng vĩnh viễn**, vì hệ cố ý không gửi mail và chưa có SLA duyệt.

Đối xứng với nó, thu hồi thu nhập khi quarantine có **hai mức theo ai ra phán quyết**:

| Ai | Thu hồi | Vì sao |
|---|---|---|
| **Operator** (người) | **tất cả**, kể cả phần đã mở khoá | Nếu không, kẻ farm chỉ cần chờ hết hạn giữ |
| **Quét tự động** (máy) | chỉ đóng băng `pending` | Dương-tính-giả không được cướp tiền người vô tội |

Và `reversed` **thôi là trạng thái chết, nhưng chỉ theo một chiều**: phán quyết của **máy** đảo
ngược được khi quarantine được gỡ; của **người** thì không. Một CHECK constraint
(`status='reversed'` **phải** kèm `reversedSource`) đảm bảo mọi khoản thu hồi từ nay **luôn trả lời
được "ai phán quyết"** — tức luôn có đường hoàn nguyên đúng thẩm quyền.

### 9.5 Blocklist là dữ liệu vận hành, không phải mã

> Ứng cứu sự cố **không còn phải deploy** (ADR 0034).

`pnpm blocklist` thêm/bớt mục chặn toàn cục ngay trên prod. Feed threat-intel công khai được nạp
tự động theo chu kỳ, và `GATE-FEEDS` đo **cả chiều âm**: feed **không** được chặn URL sạch, và mục
của operator/seed **sống sót** qua một lượt sync.

---

## 10. Open-core: seam là một ràng buộc pháp lý

### 10.1 Bốn lớp chế độ

| Lớp | Chế độ | Gồm |
|---|---|---|
| **Core** | **AGPLv3**, mở, self-host | Trụ 1,2,3,4,5,6,8,9 bản nền — **cả nền tảng link** |
| **`/ee`** | Commercial license | SSO/SAML · chính sách data residency · white-label agency · audit nâng cao/SLA · 9Link Ads |
| **Cloud** | **Miễn phí, không giới hạn** | Hosted, scale, edge toàn cầu |
| **Monetize network** | **Doanh thu chính** | Quảng cáo trên cổng của link, chia lại cho chủ link, giữ phần chênh + take-rate GMV |

**Quy tắc:** mở **toàn bộ** khả năng "làm link"; giữ thương mại phần "vận hành ở quy mô + kiếm tiền
+ niềm tin enterprise".

### 10.2 Vì sao seam là ràng buộc pháp lý chứ không phải thẩm mỹ

> AGPLv3 áp cho **toàn bộ tác phẩm phái sinh**. Một lần phát hành với `import "ee/…"` trong Core là
> mã thương mại **ĐÃ** được phát hành dưới AGPL — và giấy phép phát ra thì **KHÔNG THU HỒI ĐƯỢC**.
> **Không có bản vá nào cho việc đó.**

Vì vậy luật không phải "đừng import `/ee`". Luật là: **Core không bao giờ nhắc tên `ee/`**, ở bất
kỳ dạng nào, kể cả một `import()` động trong một nhánh không bao giờ chạy. Bởi vì một chuỗi đường
dẫn `ee/` nằm trong Core là **bước đầu tiên** trên con đường dẫn tới lỗi không rút lại được đó.

Cơ chế:

```
Core: instrumentation.ts ──▶ loadEeExtensions()  (từ "@/lib/ee-entry")
                                  │
                   ┌──────────────┴──────────────┐
      OSS (không có ee/)                  Cloud (có ee/)
 lib/ee-entry.ts — HÀM RỖNG          ee/entry.ts ──▶ register*(...)
      nhưng VẪN SEAL 5 ổ                  rồi SEAL trong finally
```

Chỗ **duy nhất** trong toàn bộ Core nhắc tới `ee` là **một dòng** trong `next.config.mjs`, nhìn thấy
được trong review, và **bị cổng đo**.

### 10.3 Ba quyết định triển khai, mỗi cái một bài học

| Quyết định | Bài học |
|---|---|
| Chuyển mạch bằng **sự tồn tại của file trên đĩa**, không bằng cờ env | Cờ env là công tắc sửa-được-ở-một-chỗ, và tiền lệ `M14_INVARIANT_BASE` cho thấy công tắc kiểu đó rốt cuộc được dùng để **TẮT CỔNG** chứ không để cấu hình |
| Dùng `NormalModuleReplacementPlugin`, **không** dùng `resolve.alias` | Đã đo: Next đăng ký `JsConfigPathsPlugin` để phân giải `@/*` từ `tsconfig.paths`, và plugin đó **thắng** alias ⇒ bundle Cloud vẫn nhận hàm rỗng trong khi **build XANH và không một dòng log nào nói gì** |
| Cổng **đo bundle**, không tin cấu hình | `scripts/gate-ee-dir.mjs` kiểm bundle OSS không chứa mã thương mại. Một cổng đọc cấu hình là một cổng tin vào thứ nó đang kiểm tra |

### 10.4 Năm ổ cắm, và vì sao cả năm đều seal

| # | Ổ cắm | Chiếm được nó sau boot nghĩa là gì |
|---|---|---|
| ① | `PaymentRail` | Đổi **đường tiền** của cả instance |
| ② | `StripeEventHandler` | Xử lý một loại webhook tiền ngoài Core |
| ③ | `SsoProvider` | **Một cửa đăng nhập lạ mở giữa đêm** |
| ④ | `ResidencyRouter` | **Đổi được nơi Event của khách đi tới** — mà mọi cổng vẫn xanh, vì số liệu vẫn hiện ra bình thường ở cụm mới |
| ⑤ | `AdNetwork` | **Chèn mã tuỳ ý lên mọi cổng đang sống, cho mọi visitor vô danh** |

Luật: **ổ trống sau boot là ổ ĐÓNG**. Bản OSS không có gì để cắm, nhưng **vẫn seal cả năm** — vì
không seal thì bất kỳ mã nào chạy trong process (một dependency bị nhiễm, một plugin tương lai) vẫn
chiếm được ổ tiền/auth/residency lúc runtime, **trên chính bản OSS**. Seal nằm trong `finally` ở
**mọi** nhánh, kể cả nhánh ném.

### 10.5 Luật kế thừa: ổ có chủ thì chủ giữ ổ, kể cả khi trả `null`

```ts
export function resolveAdNetwork(): AdNetwork | null {
  if (adNetworkSlotTaken()) return registeredAdNetwork();   // ← kể cả null
  const html = process.env.AD_NETWORK_TAG ?? "";
  return html.trim() !== "" ? new EnvAdNetwork(html) : null;
}
```

Bản ngây thơ sẽ viết `registeredAdNetwork() ?? envNetwork()`. Trên Cloud, `/ee` trả `null` nghĩa là
*"publisher này **chưa được duyệt** vào mạng"* — rơi xuống `EnvAdNetwork` nghĩa là phục vụ bằng mã
quảng cáo trong env của **chính instance**, doanh thu **về túi ta**, **không một dòng ledger nào**,
và mọi cổng vẫn xanh.

### 10.6 Cấu hình nửa vời phải chết lúc boot

`takeRateBps()`, `eeAdRates()`, `createEeResidencyRouter()`, `createSamlSsoProvider()` đều được
kiểm/gọi **ngay trong `loadEeExtensions()`** dù chưa dùng tới. Không kiểm lúc boot thì lỗi cấu hình
nổ ở **request đầu tiên của một khách hàng thật đang định trả tiền** — fail-closed đúng, nhưng phát
hiện **muộn nhất có thể**. Với residency, phát hiện muộn = **hợp đồng đã bị vi phạm** trong khoảng
thời gian đó.

**Server chết lúc khởi động là tín hiệu ồn ào và đúng** cho một instance nhận tiền mà không biết
mình thu bao nhiêu.

---

## 11. Thị trường và lộ trình

### 11.1 Ba lớp

| Lớp | Khung thời gian | Trụ | Khách | Doanh thu |
|---|---|---|---|---|
| **LAND** | 0–6 tháng | 1, 2, 3, 5, 8 | B2B mid-market: brand, agency, marketer | ~~SaaS~~ → **quảng cáo trên link** + white-label `/ee` |
| **EXPAND** | 6–15 tháng | + 4, 7, 9 | Creator/KOC toàn cầu | GMV take-rate + marketplace 2 chiều brand↔creator |
| **PLATFORM** | 15+ tháng | + 6 | Enterprise / Government (MENA + US) | `/ee` + self-host + residency |

**Cả ba lớp đã đóng về mặt mã**: 9/9 trụ sống. Lớp còn lại là **thị trường**, không phải kỹ thuật.

### 11.2 Wedge

> **B2B "Link Intelligence"** (hợp nhất short + deeplink + QR + attribution) · beachhead **MENA**
> (RTL/Arabic + WhatsApp + data residency UAE) → scale và uy tín tại **US**.

Vì sao MENA trước, dù US lớn hơn:

1. **Ba yêu cầu của MENA là ba thứ đối thủ đóng không có** (§8.6) — và chúng **không thể thêm về
   sau** mà không viết lại kiến trúc: residency là quyết định về *nơi dữ liệu nằm*, không phải một
   tuỳ chọn export.
2. Sau pivot, đường vào **không phải** một sản phẩm giá rẻ hơn — nó là một sản phẩm **miễn phí không
   giới hạn**. Ở một thị trường mà chi phí là rào cản chính của công cụ link, đây là một lời chào
   khó từ chối.
3. Uy tín "chủ quyền dữ liệu" ở MENA là **tài sản bán được ở US** cho phân khúc regulated
   (healthcare, finance, public sector) — chiều ngược lại thì không.

### 11.3 Ba chỉ số sống còn

| # | Chỉ số | Vì sao nó, không phải MRR |
|---|---|---|
| 1 | **Số công cụ khách "gỡ bỏ" nhờ 9Link** | Đo **giá trị hợp nhất** — luận điểm §2. Nếu khách không gỡ được công cụ nào, chúng tôi chỉ là một công cụ thứ sáu |
| 2 | **ARPU theo vùng** | Kỳ vọng Gulf > US > còn lại. Đây là phép kiểm trực tiếp của giả thuyết wedge |
| 3 | **% doanh thu qua white-label agency** | Agency là kênh nhân bản: một agency = N brand. Đây là đòn bẩy, không phải doanh thu |

### 11.4 Vấn đề khởi động nguội của mạng quảng cáo — và cách nó được giải

Ngã rẽ ① (9Link **tự làm mạng**, không chỉ cắm mạng ngoài) có một sự thật khó chịu:

```
chưa có advertiser → ô quảng cáo rỗng → publisher kiếm 0đ → không ai ở lại
      → không có lưu lượng để chào advertiser → chưa có advertiser
```

Cắm mạng bên thứ ba thì fill rate ~100% từ giờ đầu. Tự làm mạng thì chuỗi nhân quả **ngược lại**.

Giải pháp là một **thang fallback bốn bậc**, nằm **trong** mạng `/ee` (không phải một chuỗi ổ cắm ở
Core — xem §10.5):

```
① campaign nội bộ khớp targeting   (doanh thu trực tiếp, biên cao nhất)
② tag mạng đối tác cho ô không bán được  (fill phần dư)
③ house ad của 9Link               (tự quảng cáo mình)
④ ""                               (khung giữ chỗ trung tính)
```

> **Ngày cắm mạng đối tác vào bước ②, tiền chảy được ngay** trong khi đội bán vẫn đi ký advertiser.
> Đây là cách duy nhất ngã rẽ ① không phải chờ hàng tháng mới có đồng đầu tiên.

Bậc ④ không làm hỏng gì — `slotHtml()` trả `""` là đường đã có sẵn trong Core — nhưng nó cũng
**không trả tiền cho ai**. Và đó chính xác là lý do §7.5 ràng buộc: **không có bảng giá thì không
bật cổng**, vì bật cổng khi chưa có CPM là bắt người xem đi 2 chặng, chờ 5 giây, để đổi lấy **0
đồng** cho chủ link và 0 đồng cho ta. Cái giá có thật (tỷ lệ bỏ đi, uy tín domain dùng chung),
phần thu thì không.

### 11.5 So sánh trung thực — và ba thứ nó cố ý KHÔNG làm

> **Ngày đo:** 2026-11-10 · **cây 9Link:** `4afa314` · **công cụ:** `node scripts/repo-measure.mjs .`
> **Nguồn số của 8 repo anh em:** `survey.json` lượt `2026-11-10T10:38:36Z` — **mỗi hàng mang SHA của
> chính nó**, vì chín hàng là chín cây khác nhau đo ở chín thời điểm khác nhau trong cùng một lượt chạy.
> **Định nghĩa của từng chiều** nằm ở `docs/so-sanh-cac-repo.md §2.2` và cố ý
> **không chép lại ở đây**: một định nghĩa tồn tại ở hai chỗ là một định nghĩa sớm muộn cũng cãi nhau.

Ba thứ bảng này **cố ý không làm**, mỗi cái là một quyết định có lý do chứ không phải thiếu dữ liệu:

1. **Không nêu một con số nào của đối thủ.** Repo này đã LOẠI trang so sánh đối thủ khỏi sản phẩm
   (`docs/UI-SPEC.md §16.4` · `docs/M14-PLAN.md §1.4`) vì *số liệu đối thủ
   mục nát theo tháng* và mang rủi ro pháp lý. Tài liệu này không tự cho mình một ngoại lệ mà trang
   sản phẩm không có.
2. **Không kết luận một tính năng nào của 8 repo anh em.** Cả 8 đang trả `403` cho phiên đo
   (`Resource not accessible by integration`), có đối chứng `404` cho một repo không tồn tại — tức *repo có
   thật, chỉ là phiên này không được cấp quyền*. Thứ đo được là **18 chiều cấu trúc**, không phải một
   kiểm kê tính năng. Ô nào cần đọc mã thì ghi `*(chưa đo được)*`.
3. **Không để một ô nào trống im lặng.** "Chưa đo được" khác "đo được và bằng 0", và gộp hai cái đó
   vào một ô trống là cách một bảng so sánh bắt đầu nói dối.

#### Bảng A — White-label: lời hứa và mã

Cột "bằng chứng" là đường dẫn thật, đo tại `4afa314` ngày 2026-11-10.

| Vế | Trạng thái | Bằng chứng |
|---|---|---|
| Cờ gói `whiteLabel` | ✅ **có**, nhưng là cờ **hiển thị** | `apps/web/lib/billing/plans.ts:48` — `free=false · pro=false · business=true`; comment dòng 36–37 khai *"cờ khả năng (boolean) — hiển thị bảng so sánh, KHÔNG đếm quota"*; test `plans.test.ts:78` khẳng định `free` = `false` |
| Ổ cắm `/ee` cho white-label | ❌ **không có ổ nào** | `ee/entry.ts` → `loadEeExtensions()` đăng ký đúng **5** ổ: `PaymentRail` · `StripeEventHandler` · `SsoProvider` · `ResidencyRouter` · `AdNetwork` (bảng ở §10.4). Không có ổ branding/white-label |
| Mã white-label trong `ee/` | ❌ **0 dòng** | `git grep -i "white.label" ee/` → đúng **2** kết quả và cả hai là **văn xuôi**: `ee/LICENSE:12` và `ee/README.md:18`. Quy mô `ee/` để đối chiếu: **50** file · **9.642** dòng TypeScript |
| Ẩn badge "Made with 9Link" trên bio | ⚪ **cơ chế render có, quyết định bật thì chưa** | `apps/web/app/[key]/route.ts:151` hardcode `showBadge: true`, kèm comment *"ẩn badge = plan-flag ADR 0015, cần plan trong hot path → defer"*. Nhánh `false` **có thật và có test**: `apps/web/lib/publish/render.ts:328` · `render.test.ts:320` |
| Branding/domain agency, sub-workspace | ❌ **mô tả phạm vi, chưa có mã** | `ee/README.md:18` — hàng `White-label agency`, **cột Core là `—`**, cột `/ee` là câu chữ mô tả |
| Đối chiếu: hai cờ `/ee` còn lại | ✅ **có mã thật** | residency `ee/govern/residency.ts` **202** dòng + **304** dòng test; SSO `ee/sso/saml.ts` **193** dòng ⇒ **hai trong ba cờ có mã, một cờ không có dòng nào** |
| Trang giá có hứa quá không | ✅ **không** | `apps/web/app/pricing/page.tsx:88` xếp `whiteLabel` vào `EE_ROWS`; `:197` trả `noCell` cho cột self-host ⇒ trang giá **không** nói bản tự host có white-label |

**Câu duy nhất về đối thủ trong toàn bộ tài liệu này**, và nó được dẫn đúng nguyên trạng:
`docs/DECISIONS.md:719` ghi rằng giá gói Business từng là **99 US$** với lý do viết
ngay tại chỗ là *"Business đã gồm white-label + data residency — những thứ đối thủ để ở bậc liên hệ"*,
và đã bị hạ xuống **59 US$** vì *định giá cao dựa trên năng lực chưa tồn tại*.

> ⚠️ **Đó là lời tự khai của repo, không phải một phép đo của phiên này.** Phiên làm việc sinh ra
> mục 11.5 không có đường ra internet tới sản phẩm của bất kỳ nhà cung cấp nào, nên **không kiểm
> chứng độc lập được** câu "đối thủ để ở bậc liên hệ". Nó được dẫn vì nó là **quyết định đã ra** và
> vì nó nói lên một điều quan trọng hơn cả tình trạng của đối thủ: **dự án này đã từng tự bắt mình
> định giá trên một năng lực chưa có, và đã sửa.** White-label chính là năng lực đó, và nó **vẫn**
> chưa có mã — xem §12.6.

#### Bảng B — Tám repo anh em: chỗ nào đo được là họ đi trước

Chỉ những chiều **đo được bằng số**. "Đọc được gì" và "chưa đọc được gì" tách ra hai cột riêng, vì
một con số cấu trúc **không** phải là một kết luận về tính năng.

| Chiều | 9Link @ `4afa314` | Repo anh em (SHA của chính nó) | Đọc được · và **chưa** đọc được |
|---|---|---|---|
| Phiên bản khung | `next@15.1.6` · `react@19.0.0` · `prisma@6.2.1` · `node >=20` | **`9Link-V2` @ `c97f082`**: `next@16.3.4` · `react@19.2.8` · `prisma@7.10.0` · `node >=22` | ✅ đọc được: **phiên bản khai trong `package.json`**. ❌ chưa: V2 có **chạy được** trên các phiên bản đó không — không đọc được mã, không chạy được build |
| Cách tách gói | `packages = db · n8n-nodes-9link · sdk`; mã tiền/cổng nằm trong `apps/web/lib/**` | **`9Link-V2`**: `packages = db · gate · ledger · networks`, `apps = edge · web · worker` | ✅ đọc được: **tên** gói. ❌ chưa: ruột từng gói, và ranh giới import có được **gác bằng cổng** không (9Link gác bằng `scripts/gate-ee-dir.mjs`) |
| Kênh đẩy (SSE/WebSocket) | **0** file mã sản phẩm | **`9do` @ `4dbc2ff`** và **`olacity` @ `b18d49d`**: mỗi repo **2** file (`apps/api/src/common/interceptors/response.interceptor.ts` · `apps/web/lib/hooks.ts`) | ✅ đọc được: **số file khớp mẫu** `EventSource` / `text/event-stream` / `WebSocket` / `socket.io`. ❌ chưa: đó là SSE thật hay chỉ một interceptor đặt header. Xem đề xuất **P-3** — chữ "real-time" đang có **hai nghĩa** trong chính tài liệu này |
| Object storage · mail cục bộ | compose của 9Link **không có** `minio`, **không có** `mailhog` | **`9do`** + **`olacity`**: engine `postgres + redis + minio`; compose gốc 4 service có cả `minio` và `mailhog` | ✅ đọc được: **tên service** trong khối `services:` (định nghĩa §2.4). ❌ chưa: có tính năng upload thật dùng `minio` hay chỉ là một service được khai |
| Số app / hình dạng bề mặt | **1** app (`web`) · **27** `route.ts` · **28** `page.tsx` | **`9do`** + **`olacity`**: **4** app (`admin · api · business · web`) · 0 route Next · 67/68 page. **`9Link-new` @ `0995bd2`**: **5** app (`api · monitor · scanner · web · worker`) + `packages = contracts · domain` | ✅ đọc được: **tên app**. `monitor` và `scanner` là hai app mà 9Link **không có hình dạng tương đương** — 9Link quét đích lúc ghi (write-time, ADR 0012) trong cùng tiến trình web. ❌ chưa: tách app riêng là **tốt hơn** hay chỉ **khác** |
| Độ sâu data layer | **34** model · **55** migration | **`9do`**: **73** model · 19 migration. **`olacity`**: **69** model · 16 migration | ⚠️ **Không đọc "nhiều model hơn = tốt hơn".** Đây là ba miền dữ liệu khác nhau; con số chỉ nói **miền của họ rộng hơn**. Chiều so đúng là *"có khái niệm nào 9Link thiếu không"* — ❌ chưa đo được, vì không đọc được `schema.prisma` của họ |
| CI | **1** workflow (`.github/workflows/atlas-ci.yml`) | **`9Link-new`** · **`9do`** · **`olacity`**: mỗi repo **2** | ✅ đọc được: **số file**. ❌ chưa: chúng **chạy gì**. Số file không phải độ phủ — chính 9Link đang mang nợ **N-1**: 47 cổng nhưng CI chỉ chạy nửa Atlas của `GATE-VNDOC` |

#### Bảng C — Chỗ 9Link đo được là đi trước, để bảng không một chiều

| Chiều | 9Link @ `4afa314` | 8 repo còn lại (đo cùng một lượt `survey.json`) |
|---|---|---|
| **LICENSE** | AGPL-3.0, **687** dòng, có carve-out `ee/` ngay đầu file | **8/8 KHÔNG CÓ FILE LICENSE** (`spdx: "KHÔNG CÓ FILE"`, `lines: 0`). Hệ quả pháp lý: mặc định là *mọi quyền được bảo lưu* — tức **không ai** được phép dùng lại mã đó, kể cả chính chủ repo muốn tái sử dụng nội bộ |
| **e2e** | **36.228** dòng / **83** file | **0** ở cả 8 |
| **Unit test** | **50.703** dòng / **240** file · tỉ lệ **0,58** | cao nhất trong 8 là `9Link-new` **8.786** dòng (tỉ lệ 0,26); `9do` 2.659 (0,05) · `olacity` 2.316 (0,04); bốn repo **0** |
| **Cổng `GATE-*`** | **47** tên phân biệt do `scripts/verify.sh` gọi | `9Link-V2` **18**; bảy repo còn lại **0** |
| **i18n** | **1.683** khoá × **3** locale (`vi · en · ar`), ba file **bằng nhau tuyệt đối**, RTL thật | **0** ở cả 8 (`i18n: null` — không phát hiện thư mục locale) |
| **C4 + ADR** | **7** file `docs/c4/` · **50** ADR | `9Link-V3` @ `35a3fef` có **7** file C4 (đó là một repo tài liệu, **450** dòng src); bảy repo còn lại **0** C4 · **0** ADR |
| **Open-core seam** | có: `ee/` **50** file + swap entry ở `apps/web/next.config.mjs` | **0/8** |

> ⚠️ **Một vế phải nói ngược lại, và nó phủ lên toàn bộ Bảng B.** Cột *"Quan hệ với 9Link"* của cả 8
> repo **vẫn là "chưa kết luận"**: `survey.json` đo được `relation.overlap_paths` (số đường dẫn trùng
> tên — `9Link-V2` **44**, `9do`/`olacity` **20**, `9Link-new` **10**) nhưng `relation.content_diff`
> là `null` cho **cả 9**. Nghĩa là ta biết *có bao nhiêu file trùng tên*, **chưa** biết *nội dung có
> giống nhau không*. Luật ④ của `docs/so-sanh-cac-repo.md §4.2` đòi chỉ kết
> luận quan hệ sau khi so `package.json` + cây thư mục + **nội dung vài file trùng tên**. Mọi đề xuất
> kiểu "lấy mã từ repo anh em" phải chờ chiều đó — vì AGPL-3.0 là **ràng buộc lan**, tái sử dụng mã
> giữa các repo là một quyết định pháp lý chứ không phải một thao tác `cp`.

#### Đề xuất — mỗi cái một bằng chứng, một giá, và một điều kiện phải có trước

| # | Đề xuất | Bằng chứng | Giá · điều kiện phải có trước |
|---|---|---|---|
| **P-1** | **Đo `relation.content_diff` trước khi nói bất cứ câu nào về tái sử dụng mã** | `relation.content_diff = null` cho cả 9 hàng của `survey.json` | Cần quyền đọc 8 repo (đang `403`). Đề xuất này **đứng đầu** vì nó là điều kiện của mọi đề xuất còn lại |
| **P-2** | **Đánh giá nâng khung theo hướng `9Link-V2` đang khai** (`next` 15→16 · `react` 19.0→19.2 · `prisma` 6→7 · `node` 20→22) | Bảng B, dòng 1 | **Không** nâng với lý do "repo anh em đã nâng". Điều kiện: một lượt `verify.sh` **XANH** trên nhánh nâng cấp. `prisma` 6→7 là **major**, và 47 cổng + 240 file unit test là tài sản đắt nhất repo này — nâng khung mà mất cổng thì đổi một phiên bản lấy một lỗ hổng |
| **P-3** | **Chốt định nghĩa chữ "real-time" trước khi so nó với bất kỳ ai** | §4 hàng trụ 5 viết *"analytics real-time"*; `README.md:339` giải nghĩa đó là **đọc thẳng ClickHouse mỗi truy vấn**; còn bộ đo §2.2 đếm **kênh đẩy** ⇒ cùng một repo ra **hai** câu trả lời ("có" và "0") | Đây đúng là bệnh mà §2 của hub sinh ra để chữa: *mỗi repo đo một kiểu thì bảng so sánh vô nghĩa*. Rẻ nhất là **một dòng định nghĩa** trong §4, không phải một dòng mã |
| **P-4** | **Chỉ cân nhắc `minio` khi có một tính năng cần object storage thật** | Bảng B, dòng 4 | Thêm một service là thêm bề mặt vận hành cho người self-host. Nó phá lời hứa *"docker compose up là chạy"* nếu không kèm tài liệu và một cổng — và 9Link hiện **không có** tính năng nào cần nó |
| **P-5** | **Cấp LICENSE cho 8 repo anh em** | Bảng C, dòng 1: 8/8 không có file LICENSE | Đây là **quyết định pháp lý của chủ dự án**, không phải của agent. Nhưng nó đang chặn mọi đường tái sử dụng — kể cả tái sử dụng nội bộ — và nó khiến mọi con số "quan hệ" ở Bảng B không thể dẫn tới hành động |
| **P-6** | **Sinh §14 bằng máy thay vì chép tay** | §14 lệch **3** con số so với `repo-measure.mjs` chạy trên **cùng một cây** — xem §12.7 dòng 8 | Công cụ đã in ra đúng các số đó. Một bảng số chép tay đặt cạnh một công cụ đo là một bảng **sẽ** trôi; §3.1 của hub đã chữa đúng bệnh này bằng marker máy sinh |
| **P-7** | **White-label: hoặc dựng, hoặc ngừng đếm nó là doanh thu** | §11.1 xếp white-label vào doanh thu lớp LAND; §11.3 chọn *"phần trăm doanh thu qua white-label agency"* làm **chỉ số sống còn số 3**; mã = **0 dòng** | Xem §12.6. Đây là đề xuất **duy nhất trong bảng không cần một quyền truy cập nào** — nó chỉ cần một quyết định |

---

## 12. Những gì chưa làm — nói thẳng

Mục này là một phần của white paper, không phải phụ lục. Một đối tác enterprise đánh giá nhà cung
cấp bằng **độ chênh giữa lời hứa và mã**, nên chúng tôi công bố cả hai.

### 12.1 Chuỗi tiền đang inert trên production

| Sự thật | Số |
|---|---|
| `ad_rate_cards` trên prod | **0 hàng** |
| `ad_earnings` trên prod | **0 dòng** |
| `ad_creatives` trên prod | **0** (`AD_NETWORK_9LINK_CREATIVE_IMG_ORIGINS` chưa cấp) |
| `ad_network_revenue` | chưa có báo cáo thật ⇒ **mọi ngày = `insufficient`** |

⇒ Cơ chế đã lên, **chưa từng chạy thật một lần nào**. Đây là trạng thái **đúng thiết kế**: thiếu
cấu hình ⇒ không hứa tiền với ai. Nhưng nó cũng có nghĩa **chưa có bằng chứng thực địa** về doanh
thu trên mỗi lượt.

> **Bản công khai lược mục này.** Lý do: chi tiết vận hành và HAI lỗ hổng còn đang mở trên production (xoay header proxy để đốt biên, và một van chống lạm dụng đang đo-mà-không-chặn). Công bố nguyên văn là chỉ chỗ cho kẻ tấn công.
> Bản đầy đủ nằm trong repo private của dự án; phần bị lược là chi tiết vận hành, không phải phần sản phẩm.

### 12.4 Kiến trúc đích chưa đạt

| | Đích đã viết | Hiện trạng |
|---|---|---|
| Lớp EDGE | Cloudflare Workers, PoP US/UAE/EU, <50ms | **Next.js route handler** sau reverse proxy. `CLOUDFLARE_*` có trong `.env.example`, **0 tham chiếu trong mã** |
| Residency prod | Đa cụm US + UAE + EU | **Một cụm** |
| Đa vùng Postgres | Hình dạng B (mỗi vùng một deployment) | Cơ chế có, **chưa triển khai** |

⚠️ Điều **vẫn đúng**: bất biến §0.1/§0.3 thật sự đòi không phải "chạy trên Workers" mà là *"đường
302 không thêm một lượt I/O nào cho mỗi tính năng mới"*. Lời hứa đó **đang được giữ và được đo**
qua 6 ngoại lệ hẹp, mỗi cái một ADR.

### 12.5 Nợ đã ghi, chưa trả

`docs/BLOCKERS.md` giữ sổ nợ công khai. Những mục đáng chú ý với một đối tác:

- Bất đối xứng thu hồi link/page · không có đường hoàn nguyên cho khoản **operator** đã thu hồi ·
  dòng `reversedSource = NULL` (trước M25) **không hoàn nguyên được**.
- Guard tràn 2^53 khi mở bề mặt rút tiền: trần thật của tiền vẫn là **2^53**, không phải trần
  `int8` — tầng tính là số học `Number`. BigInt mua **độ bền lược đồ**, không mua thêm dải số.
- Ngân sách campaign dùng chung xuyên cụm (hôm nay một advertiser chạy hai vùng được dựng **hai
  lần**, ngân sách tách đôi).
- Hai lượt rà (bảo mật T25.8, a11y T25.6) **do chính tác giả diff thực hiện** — điểm mù của bản
  thiết kế không tự lộ ra. Ghi là **việc chưa làm**, không phải việc đã xong.

### 12.6 White-label: một nguồn thu được công bố, chưa có một dòng mã

Mục này tồn tại vì §11.5 Bảng A đo ra một khoảng cách, và khoảng cách đó **nằm trong chính tài liệu
này** chứ không nằm ở đâu khác.

| | |
|---|---|
| White-label xuất hiện trong white paper ở | **5** chỗ: §6.1 (nguồn thu phụ của `/ee`) · §6.2 (hàng "Giấy phép `/ee`") · §10.1 (bảng bốn lớp chế độ) · §11.1 (doanh thu lớp **LAND**) · §11.3 (**chỉ số sống còn số 3**) |
| Số dòng mã white-label trong `ee/` | **0**. `git grep -i "white.label" ee/` → **2** kết quả, cả hai là văn xuôi (`ee/LICENSE:12` · `ee/README.md:18`) |
| Số ổ cắm `/ee` dành cho nó | **0** trong **5** ổ (§10.4) |
| Hai cờ `/ee` còn lại | **có mã thật**: residency 202 dòng + 304 dòng test · SSO/SAML 193 dòng |

⇒ **Một trong ba năng lực `/ee` được công bố là có mã; hai năng lực có thật.** Hàng §6.2
*"Giấy phép `/ee` — ✅ cơ chế, chưa có khách"* gộp cả ba vào một dấu ✅, và đó là một câu **rộng hơn
mã** cho đúng một vế. Bản 1.1 tách vế đó ra.

**Vì sao chuyện này đáng một mục riêng, không phải một chú thích cuối trang:**

1. **Nó đã từng được dùng để định giá.** `docs/DECISIONS.md:719` ghi lại việc hạ giá Business từ
   99 US$ xuống 59 US$ chính vì lý do *"Business đã gồm white-label + data residency"* là định giá
   trên năng lực **chưa tồn tại**. Quyết định đó đúng. Điều chưa xảy ra là **phần còn lại của tài
   liệu** chưa được sửa cho khớp với nó.
2. **Nó đang là một chỉ số sống còn.** §11.3 chọn *"phần trăm doanh thu qua white-label agency"* làm
   chỉ số số 3 để đo đòn bẩy kênh. Một chỉ số đo trên một năng lực chưa có mã sẽ **bằng 0 vĩnh viễn**
   và trông như một thất bại về bán hàng, trong khi nó là một khoảng trống về thi công.
3. **Chiều hỏng của nó là chiều im lặng.** Không có cổng nào đỏ vì thiếu white-label: `plans.test.ts`
   khẳng định `free.whiteLabel === false` (đúng), `/pricing` hiển thị nó như một năng lực `/ee`
   (đúng), và bio page hardcode `showBadge: true` (đúng theo ADR 0015, vốn **defer** chứ không hứa).
   Mọi thứ đều xanh. Thứ duy nhất sai là **ấn tượng** mà năm chỗ trong tài liệu này tạo ra.

**Hai đường ra, và đây là quyết định của chủ dự án, không phải của tài liệu:**

| | Đường | Giá |
|---|---|---|
| **(a)** | **Dựng nó.** Bắt đầu bằng thứ rẻ nhất đã có sẵn cơ chế: ẩn badge theo cờ gói (`render.ts:328` đã có nhánh `false` và đã có test), rồi tới ổ cắm thứ sáu cho branding agency | Đụng hot path `/{key}` — mà hot path **không mang plan** theo thiết kế (comment tại `route.ts:150`). Nên bước đầu tiên không phải mã, nó là **một ADR** trả lời câu: *plan đi vào đường 302 bằng cách nào mà không thêm một lượt I/O* |
| **(b)** | **Ngừng đếm nó là doanh thu** cho tới khi có mã: sửa §11.1 và §11.3, giữ white-label ở §10.1 như **phạm vi đã tuyên bố** của `/ee` | Rẻ, và không làm tài liệu nói dối. Cái giá là một câu chuyện bán hàng ngắn hơn |

Bản 1.1 **chọn (b) về mặt câu chữ** — tức là mục này và §11.5 Bảng A công bố trạng thái thật — và
**để ngỏ (a) về mặt quyết định**, vì dựng hay không dựng là việc của chủ dự án.

### 12.7 Những quyết định còn treo — cần chữ ký, không cần mã

Khác §12.2 (thiếu **cấu hình**) và §12.5 (nợ **thi công**), mục này liệt kê những chỗ mà **mã đã
đúng** và thứ thiếu là **một câu trả lời**. Mỗi dòng ghi rõ ai chốt, vì một việc không có tên người
chốt là một việc sẽ không xảy ra.

| # | Treo gì | Bằng chứng (đo tại `4afa314`, 2026-11-10) | Ai chốt |
|---|---|---|---|
| 1 | **ADR 0049** — chọn LangGraph.js thay MetaGPT | Cả hai file ADR ghi nguyên văn: `Trạng thái: Đề xuất (2026-11-08) — **chưa chấp nhận**. Cần chữ ký chủ dự án.` | Chủ dự án |
| 2 | **ADR 0050** — giữ runtime dài hạn tự host, không di trú sang Vercel | như trên; và nó là bản chốt của `docs/agentworks/06-kien-truc-chot.md` ⇒ **cả một nhánh tài liệu đang treo trên một ADR chưa ký** | Chủ dự án |
| 3 | **ADR 0048** — còn hiệu lực tới đâu | `⚠️ CHẤP NHẬN MỘT PHẦN — bị ADR 0049 thay thế một phần`. ADR 0050 nói rõ *"vẫn ở trạng thái chấp nhận một phần, không đổi"* — tức trạng thái lưng chừng này **đã được chấp nhận là tạm**, nhưng chưa ai chốt nó là **vĩnh viễn** hay chờ 0049 được ký | Chủ dự án |
| 4 | **Bảy ẩn số M-1…M-7** | bảng ở cuối `docs/adr/0050-giu-runtime-dai-han-tu-host.md`. Hai cái chặn đường: **M-1** Caddyfile **không nằm trong repo** (`find -iname "*Caddyfile*"` = 0) ⇒ không review được cấu hình đệm cho SSE; **M-4** host hiện tại có đủ RAM/CPU cho +2 container không — ADR tự ghi *"đo trước khi hứa"* | M-1: chủ dự án (vượt phạm vi AgentWorks) · M-4: người giữ host · M-2/M-3: người dựng A1/A3 · M-7: **chưa thiết kế** |
| 5 | **12 việc `[human]`** | §12.2 — không chép lại ở đây, vì một danh sách tồn tại ở hai chỗ là một danh sách sớm muộn cũng cãi nhau. Nguồn duy nhất: `docs/BLOCKERS.md` dòng đầu | Chủ dự án |
| 6 | **Ba cầu dao "chỉ ĐO"** | §12.3 — cả ba treo trên **cùng** dòng 2/3 của §12.2 | Chủ dự án |
| 7 | **Node n8n: đường A, B hay C** | `docs/connect/N8N-LICENSE-REPORT.md` §2–§3. Báo cáo tự ghi: *"đây là quyết định về TÀI SẢN, xin đừng để một agent tự quyết"*, và **kiến nghị đường A** kèm ba lý do. Kèm **hai vế phụ chưa ai trả lời**: (i) `CLAUDE.md:32` viết *"Tất cả AGPL/MIT/Apache — không license độc quyền"* trong khi **cả ba** đường đều kéo `n8n-workflow` (Sustainable Use License) vào `devDependencies` ⇒ đây là **nới một luật cứng**, cần một câu "có" tường minh; (ii) node nộp verification sau 2026-05-01 phải phát hành qua GitHub Actions + npm provenance, mà `site.repoUrl` đang `null` vì chủ dự án đã chốt 2026-08-03 **giữ repo riêng tư** ⇒ đường B vướng thêm một lần nữa, **độc lập** với giấy phép | Chủ dự án |
| 8 | **Ba con số ở §14 đã trôi — sửa thì dễ, quyết định là có sinh bằng máy không** | Đo bằng chính công cụ của repo (`node scripts/repo-measure.mjs .`) trên **cùng một cây**: ADR khai **47** → thật **50**; cổng `GATE-*` khai **46** → thật **47**; file e2e khai **78** → thật **83**. Bản 1.1 sửa cả ba và **ghi kèm định nghĩa**, vì con 78 hoá ra là `ls apps/web/e2e | wc -l` (mọi đuôi file, một thư mục) còn 83 là số file **nguồn** dưới mọi thư mục `e2e/` — hai định nghĩa khác nhau trong **cùng một bảng** | Đề xuất **P-6** (§11.5). Chủ dự án chốt: sinh bằng máy, hay tiếp tục chép tay và chấp nhận nó sẽ trôi lần thứ tư |
| 9 | **Phạm vi CI** | `.github/workflows/atlas-ci.yml` khai `paths:` gồm `docs/atlas/**` · `scripts/check-atlas.mjs` · `scripts/check-vn-docs.mjs` · `scripts/publish-atlas.sh` · `publish/atlas-site/**` · và chính nó. **Không có** `scripts/survey-9repo.sh` · `survey-to-md.mjs` · `survey-txt-to-json.mjs` ⇒ sửa bộ đo 18 chiều **không** kích hoạt CI. Đây là nợ **N-1** ở `docs/so-sanh-cac-repo.md §3.2`: *47 cổng nhưng CI chỉ chạy nửa Atlas của `GATE-VNDOC`* | Chủ dự án — vì `.github/` nằm trong `denyPaths` (`docs/agentworks/02-kien-truc.md:188`), agent sửa nó cần một **ngoại lệ được gọi tên** |
| 10 | **White-label** | §12.6 | Chủ dự án |
| 11 | **Bản tiếng Anh đang lệch bản tiếng Việt** | dòng đầu của `WHITEPAPER.en.md` tự khai lệch kể từ bản 1.1 | Ai sửa bản EN kế tiếp |
| 12 | **Phép lược bản công khai đang KHÔNG nhất quán với trang Atlas** | Bản 1.1 lược §12.2–§12.3 khỏi bản white paper công khai. Nhưng `docs/atlas/index.html` — **đã đăng công khai** ở repo `khanhlegood3/9link-atlas` từ commit `8068ca1` — chép lại **nguyên văn** đúng hai lỗ hổng đó, kèm cả phép đo `899 lọt / 1 chặn` (dòng 1117 của trang), và `publish/atlas-site/README.md` tự khai đây là *"một quyết định công bố có ý thức"*. ⇒ **Lược white paper không đóng lỗ đó**; nó chỉ tránh tạo ra bản sao công khai THỨ HAI. Cổng của bản lược vẫn gác đúng phạm vi của nó, và sự không nhất quán này được ghi ở đây thay vì để người đọc tự phát hiện | Chủ dự án — hoặc chấp nhận cả hai đều công khai, hoặc lược **cả** trang Atlas (việc sau đụng `scripts/check-atlas.mjs`, cổng đang gác trang đó) |

---

## 13. Rủi ro và cách chúng được quản trị

| Rủi ro | Mức | Quản trị |
|---|---|---|
| **Domain dùng chung bị blacklist** | **Tồn vong** | Bốn lớp phòng thủ (§9) + blocklist là dữ liệu vận hành (không cần deploy) + link kiếm tiền quét dày gấp 4 + trang cổng có **nhãn cảnh báo + nút báo cáo** ×3 locale |
| **Uy tín domain khi bật cổng mặc định** | Cao, **đã chấp nhận có ý thức** | Người sáng lập đã chấp nhận rủi ro uy tín `9link.com`; ghi trong `docs/M36-PLAN.md §5` như một ngã rẽ **có thể phải xét lại** khi có số prod thật về tỉ lệ link bị báo cáo. Không tách domain riêng ở v1 |
| **Gian lận lượt hiển thị rút tiền thật** | Cao | Hai hệ quả tách rời (§7.3) · giữ tiền `holdDays` · dedup/velocity/bot · thu hồi hai mức theo ai phán quyết · `GATE-ADGATE` 73 phép đo + 26 đột biến chạy thật |
| **Lỗ biên im lặng** (CPM khai > eCPM thật) | Cao | `ad_network_revenue` đối chiếu theo (ngày, nước, nguồn); thiếu dữ liệu ⇒ `insufficient`, **từ chối kết luận**; `GATE-MARGIN` 49/49 |
| **Chi trùng / nhân đôi tiền** | Cao | Khoá tự nhiên 5 cột · CAS trên hàng workflow (đếm rowcount, không đọc số dư) · partial unique index "một yêu cầu mở" · `sourceWorkspaceId` NOT NULL |
| **Dữ liệu vùng rò sang cụm sai** | Cao (pháp lý) | Fail-closed (§8.3) · gác bản đồ cụm lúc boot · gác allowlist operator **trên từng cụm** · cổng đo **cả vế âm** (cụm này không thấy hàng cụm kia) |
| **Chi phí phục vụ không có doanh thu bù** | Trung bình | `serveAllowanceOverride` (§6.4) — van operator, ra đời cùng ngày gỡ trần gói |
| **Khởi động nguội của mạng quảng cáo** | Trung bình | Thang fallback 4 bậc (§11.4): cắm mạng đối tác ở bậc ② là tiền chảy ngay |
| **Phụ thuộc Stripe** | Trung bình | `PaymentRail` là một ổ cắm; `StripeInstanceRail` (BYO, 9Link lấy 0%) và `StripeConnectRail` (`/ee`) là hai impl. REST thuần, **không SDK npm**, pin api-version |
| **AGPL không tương thích với một số procurement** | Trung bình | Self-host AGPL cho ai muốn đọc mã; `/ee` commercial cho ai cần giấy phép doanh nghiệp; Cloud miễn phí cho ai không muốn vận hành |
| **"Jack of all trades"** | Thấp (đã trả xong) | Lộ trình Land → Expand → Platform **đã đi hết**; 9/9 trụ sống. Rủi ro nay là **thị trường**, không phải phạm vi |

---

## 14. Bằng chứng kỹ thuật

Một white paper về một sản phẩm mở nên đo được. Đây là những con số lấy từ cây mã, không phải ước
lượng:

| | |
|---|---|
| Mốc đã đóng | **45** (M0 → M45), lộ trình Land → Expand → Platform **hoàn tất về mã** |
| Trụ sống | **9/9** |
| Quyết định kiến trúc có ADR | **50** (`docs/adr/`) |
| Mã sản phẩm | **~108.700** dòng TypeScript/TSX (phi-test) |
| Mã kiểm thử | **~50.700** dòng (tỷ lệ ≈ 0,47 dòng test / dòng sản phẩm) |
| File unit test | **240** |
| File e2e | **83** file **nguồn** dưới mọi thư mục `e2e/` (`apps` 75 · `ee` 8) — con số **78** ở bản 1.0 là `ls apps/web/e2e`, tức mọi đuôi file trong MỘT thư mục: hai định nghĩa khác nhau nằm trong cùng một bảng |
| Cổng `GATE-*` trong `scripts/verify.sh` | **47** tên phân biệt |
| Prisma model | **34** · migration **55** |
| Khoá i18n | **1.683** × 3 locale (`vi` · `en` · `ar`, RTL thật) |
| Unit test (lượt verify gần nhất) | web **3.916** / 223 file · `/ee` **178** / 14 file |
| `tsc --noEmit` · `eslint --max-warnings=0` | **0** · **0** |

> **Hai nguồn, khai rõ để không gộp làm một.** Các dòng **LOC · file test · cổng · Prisma · i18n ·
> ADR** đo tại cây `4afa314` ngày 2026-11-10 bằng `node scripts/repo-measure.mjs .` (định nghĩa từng
> chiều: `docs/so-sanh-cac-repo.md §2.2`). Các dòng **số lượt test · `tsc` ·
> `eslint`** là của **lượt `verify.sh` gần nhất**, không phải của cùng một lượt đo — chúng được giữ
> nguyên văn từ bản 1.0 và **không** được phiên này xác minh lại, vì một lượt `verify.sh` đầy đủ cần
> Docker + Postgres và chạy ~50 phút.
>
> ⚠️ **Bản 1.0 khai 47 ADR · 46 cổng · 78 file e2e; cả ba đều đã trôi.** Sửa tay ba con số đó là
> cách làm cho chúng trôi lần thứ tư — nên đề xuất **P-6** ở §11.5 là **sinh cả bảng này bằng máy**,
> đúng khuôn mà §3.1 của hub đã áp dụng. Ghi lại ở §12.7 dòng 8 như một quyết định còn treo.

### 14.1 "Định nghĩa XONG" — sáu vế

Một mốc/tính năng chỉ "xong" khi đủ (`CLAUDE.md §6`):

1. ✅ **Test** (unit + ít nhất 1 e2e cho luồng chính) xanh
2. ✅ **`next build` XANH** — không chỉ test/lint/typecheck (bài học #5: build chết ở prerender mà
   ba cổng kia vẫn xanh)
3. ✅ **Review** đạt luật cứng dự án
4. ✅ **i18n + RTL** cho phần UI; **responsive** mobile-first
5. ✅ **Security-review** nếu chạm auth/payment/monetize/upload/data-residency
6. ✅ **Doc** cập nhật (README/CHANGELOG/API) + PROGRESS ghi mốc

### 14.2 Ba thực hành kỹ thuật đáng kể với một người đánh giá

**① Mọi cổng phải tự chứng minh nó ĐỎ được.** Một cổng chỉ có vế âm là một cổng **xanh vì nhánh
chưa bao giờ chạy**. Repo này có **bảng đột biến chạy thật** (ví dụ: 26 đột biến cho `GATE-ADGATE`,
8/8 cho T45.3b) — mỗi đột biến phải làm **đúng assert** đỏ. Ba đột biến từng sống sót, và **cả ba**
lộ ra một lỗ thật trong cổng chứ không phải trong mã.

**② Cổng phải tự ghi kết quả của mình vào log.** `verify.sh` in ba dòng
`===SERVER=` · `===SKIP=` · `===VERIFY_EXIT=`, với `_VERIFY_SKIPS` đếm **mọi** bước bị bỏ qua và
`VERIFY_REQUIRE_NO_SKIP=1` để biến "xanh nhưng chưa đủ" thành **đỏ**. Lý do: HANDOFF từng khai
"0 SKIP" trong khi **bốn cổng M45 chưa từng nằm trong suite** — `SKIP` đếm những cổng **đã ở
trong** suite, không đếm những cổng **lẽ ra phải ở đó**.

**③ Một chú thích cấm không phải một cổng.** `/security` (trang công khai, vô danh) từng import
một module kéo Prisma vào đồ thị — **ba dòng sau** một chú thích cấm đúng điều đó, và cổng gác vẫn
**xanh** vì nó chỉ so specifier **trực tiếp**. Nay cổng duyệt import **bắc cầu**. Bài học tổng quát:
*cổng đo một BƯỚC không gác được một ĐỒ THỊ*.

### 14.3 Văn hoá ghi lại sai lầm

`HANDOFF.md` (443 KB) và `docs/PROGRESS.md` (4.989 dòng) ghi **nguyên nhân gốc** của từng sự cố, kể
cả những sự cố do chính người viết tài liệu gây ra. Vài ví dụ có giá trị với bất kỳ ai sẽ vận hành
hệ này:

- *"Mẫu thử ghim ngày tuyệt đối cạnh một truy vấn có cửa sổ trượt là **mìn hẹn giờ**, và nó nổ vào
  một ngày không ai commit gì."* — `GATE-WALLET` đã chết ~8 ngày mà không ai biết, vì lịch: một
  mẫu thử ghim `2026-08-01` tự trôi ra khỏi cửa sổ 30 ngày vào 31/08.
- *"Cổng dựng xong mà không nối vào `verify.sh` thì **không tồn tại**."*
- *"Dọn bảng **trước**, nới trần **sau**. Thứ tự ngược lại sẽ mua vài tuần yên ổn rồi nổ lại ở đúng
  chỗ này với một con số to hơn."*
- *"Đừng tin chú thích mình vừa viết."* — ba khẳng định của tác giả về 2FA đều **sai khi đo**.

---

## 15. Phụ lục

### 15.1 Thuật ngữ

| Thuật ngữ | Nghĩa |
|---|---|
| **trụ** (pillar) | Một trong 9 năng lực của nền tảng |
| **Event** | Một hàng trong bảng `events` (ClickHouse). 6 loại: `click` `scan` `view` `convert` `earn` `impression` |
| **đường nóng** / hot path | Chuỗi lệnh phục vụ `GET /{key}` trước khi trả phản hồi |
| **zero-cost** | Tính năng tắt ⇒ byte-identical mốc trước + 0 lượt I/O thêm |
| **ổ cắm** (socket) | Một trong 5 registry mà `/ee` chiếm lúc boot rồi seal |
| **cổng** (gate) | Một phép đo máy (`GATE-*`) trong `verify.sh` hoặc `e2e/` |
| **quarantine** | Phán quyết **dính**, tách khỏi `flagged` dẫn xuất |
| **rail** | Ổ cắm đường tiền (`PaymentRail`) |
| **take-rate** | Phí nền tảng trên GMV, tính bằng **bps số nguyên** |
| **minor units** | Đơn vị nhỏ nhất của đồng tiền (cent). Mọi con số tiền trong hệ |
| **residency mức 1/2/3** | nhãn vùng → Event theo cụm → PII/Postgres theo cụm |
| **operator** | Người vận hành **cấp instance**, không thuộc workspace nào |

### 15.2 Tài liệu tham chiếu trong repo

| Tài liệu | Nội dung |
|---|---|
| [`docs/c4/`](c4/README.md) | **Mô hình C4 bốn mức** + 12 sơ đồ động + deployment view |
| `CLAUDE.md` | Luật bất biến §0–§9 — tài liệu gốc của mọi quyết định |
| `docs/PRD.md` | Vấn đề · giải pháp · người dùng · moat · chỉ số sống còn |
| `docs/ARCHITECTURE.md` | Sơ đồ 4 lớp + mô hình đối tượng xương sống |
| `docs/adr/` | **50 ADR** — mỗi quyết định kiến trúc, kèm các lựa chọn đã bác |
| `DECISIONS.md` · `docs/DECISIONS.md` | Quyết định kinh doanh & thi công, có ngày và người chốt |
| `docs/openapi.json` | Hợp đồng REST 3.1, sinh từ Zod |
| `docs/connect/WEBHOOKS.md` | Hợp đồng tiêu thụ webhook OUT |
| `docs/OPERATIONS-BILLING.md` | Bật billing trên bản self-host |
| `docs/BLOCKERS.md` | **Sổ nợ công khai** + danh sách `[human]` — nguồn duy nhất của §12.2 |
| `docs/so-sanh-cac-repo.md` | **18 định nghĩa đo** dùng chung cho 9 repo · 6 luật của bảng so sánh · §3.1 do máy sinh · §4.1–§4.3 ba lỗi đo đã trả giá. Nguồn của mọi con số ở §11.5 |
| `scripts/redact-whitepaper.mjs` | Bộ lược sinh ra **bản công khai** của chính tài liệu bạn đang đọc: một cặp marker `PUBLIC-REDACT` trong nguồn quyết định phần bị cắt, và bản công khai là **hàm** của nguồn chứ không phải một bản chép tay. Cổng của nó kiểm **hai chiều** — danh sách chuỗi CẤM phải vắng **và** danh sách chuỗi phải GIỮ phải còn — vì một bộ lược chỉ có danh sách CẤM thì *"xoá sạch cả tài liệu"* đạt tuyệt đối. Mọi link tương đối không sống được ở repo đích bị bẻ thành code span: đường dẫn còn đó để truy vết, không còn cú bấm nào ra 404 |
| `docs/PROGRESS.md` | Nhật ký 45 mốc |
| `README.md` | Trạng thái từng mốc + quickstart + giới hạn self-host |

### 15.3 Chạy thử trong 10 phút

```bash
# 1) Hạ tầng (Postgres 55433 · ClickHouse 8123 · Redis 56390)
docker compose -p 9link-dev -f infra/docker-compose.yml up -d

# 2) Cài + DB. `db:seed` IN RA API key demo THÔ đúng một lần
cp .env.example .env
pnpm install && pnpm db:migrate && pnpm db:seed

# 3) Web + worker
pnpm --filter web dev -- -p 3939     # dashboard http://localhost:3939
pnpm --filter web worker             # ingest Event (terminal khác)

# 4) Cổng verify (build + unit + e2e) — ~50 phút, đọc BA dòng `===`
VERIFY_REQUIRE_NO_SKIP=1 bash scripts/verify.sh
```

Bản container đầy đủ (self-host): `deploy/docker-compose.yml` — Postgres + ClickHouse + Redis +
init + web + worker, sau một reverse proxy, không publish port dữ liệu.

**Bắt buộc trước khi mở public:** `bash scripts/launch-check.sh <env-file>` phải **PASS**.

### 15.4 Giấy phép

| Phạm vi | Giấy phép |
|---|---|
| Toàn bộ Core (`apps/`, `packages/`, `infra/`, `deploy/`, `scripts/`, `docs/`) | **AGPLv3** — xem `LICENSE` |
| `ee/` | **Commercial** — xem `ee/LICENSE` |

Ranh giới là **chiều import**: `/ee → Core` được phép; `Core → /ee` **bị cấm bằng cổng máy**.

---

*Tài liệu này được viết từ mã nguồn, và mọi khẳng định trong nó đều trỏ được tới một đường dẫn
trong repo hoặc một cổng đo. Nếu bạn tìm thấy một câu nói rộng hơn mã, đó là một bug — xin mở
issue.*
