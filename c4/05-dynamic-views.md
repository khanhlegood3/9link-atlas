# Dynamic views — 12 luồng chạy qua các hộp

> **Neo số liệu — đọc trước khi tin bất kỳ con số nào ở dưới.** File này được viết ngược từ mã
> nguồn thật tại commit `b559c68` (`main`, 2026-09-08), cùng một neo với `README.md` của bộ C4.
> Mọi đường dẫn, số file, số dòng và số lượng nêu trong file **thuộc cây đó**, không phải cây mới
> nhất. Ví dụ đã đo bằng `git ls-tree`: `docs/adr/` có **47** file `.md` tại `b559c68` và **50** tại
> `4af8932` (2026-11-10) — cả hai đều đúng, tại cây của mình. Nên hãy đọc các con số ở đây như một
> **ảnh chụp có ngày**, và muốn biết hiện trạng thì đo lại trên cây đang đứng.


> **Câu hỏi tài liệu này trả lời:** các container và component ở Level 2–3 **phối hợp theo thứ tự
> nào** trong những luồng quan trọng. Đây là *supplementary view* của C4 — không phải một mức
> phóng đại thứ năm, mà là cùng những hộp đó, vẽ theo thời gian.
>
> **Vì sao tài liệu này đứng trước Level 3 trong thứ tự đọc khuyến nghị:** ở một hệ thống mà tài
> sản cốt lõi là *đường đi của một cú click*, đọc luồng trước rồi mới đọc cấu trúc khiến cấu trúc
> có nghĩa ngay.
>
> **Quy ước:** mỗi luồng ghi kèm **bất biến nó bảo vệ** và **cổng máy nào đo nó**. Một luồng không
> có cổng đo là một luồng đang được tin chứ không phải được biết.

| # | Luồng | Bất biến được bảo vệ | Cổng đo |
|---|---|---|---|
| D1 | Click thường → 302 | zero-cost | `e2e:m0-loop` · `m1-create` |
| D2 | Click có routing + A/B + tracking | rule đánh giá không I/O | `e2e:m3-route` |
| D3 | Link có mật khẩu | đích chỉ lộ sau mật khẩu | `e2e:m5-govern` |
| D4 | Qua cổng quảng cáo → impression → tiền | hai hệ quả tách rời | `GATE-ADGATE` (73 phép đo) |
| D5 | Link gắn bio page | page thắng routing, CSP không đổi | `e2e:m7-publish` · `GATE-PAGE-AD` |
| D6 | Ingest Event | một lớp dữ liệu, fail-closed residency | `GATE-M19-WRITE` · `GATE-RESIDENCY` |
| D7 | Attribution convert | không bịa ra tiền | `GATE-ORACLE` · `GATE-REPLAY-ORACLE` · `GATE-IDEM` |
| D8 | Kết sổ → ví → rút tiền | không nhân đôi, không chi trùng | `GATE-WALLET` · `GATE-PAYOUT` · `GATE-STEPUP` |
| D9 | Webhook Stripe billing | single-writer của tiền | `GATE-MONEY` · `GATE-PERIOD` |
| D10 | Webhook OUT delivery | at-least-once, không phải Event | `e2e:m9-connect` · `m13-connect-rest` |
| D11 | Chống lạm dụng | quyết định ≠ giá trị dẫn xuất | `GATE-ABUSE` · `GATE-FEEDS` |
| D12 | Boot & seal ổ cắm | không ai chiếm ổ sau boot | `GATE-EE-DIR` · `GATE-PLATFORM-DEFAULT` |

---

## D1 — Click thường: đường cơ sở mà mọi mốc phải giữ nguyên byte

```mermaid
sequenceDiagram
  autonumber
  actor V as Người xem
  participant R as app/[key]/route.ts
  participant LC as lib/link-cache.ts
  participant RD as Redis
  participant PG as Postgres
  participant EV as lib/events.ts

  V->>R: GET /{key}  (Host: go.acme.com)
  R->>R: normalizeHost() — strip port, lowercase
  R->>LC: resolveLink(domain, key)
  LC->>RD: GET linkcache:{domain}:{key}
  alt cache miss
    LC->>PG: SELECT ~20 cột TƯỜNG MINH (không SELECT *)
    LC->>RD: SET qua script Lua CAS theo links.version, TTL 60s
  end
  LC-->>R: ResolvedLink
  R->>R: isLinkRedirectable(link) — archived/expired/flagged/quarantined/serveBlocked
  Note over R: routing == null, pageId == null,<br/>adGate == null, passwordHash == null,<br/>clickLimit == null ⇒ KHÔNG một nhánh nào chạy
  R->>EV: buildClickEvent(link, req, "", "", "") — ĐỒNG BỘ, thuần CPU
  R->>R: after(() => enqueueEvent(event))
  R-->>V: 302 Location: {link.url}  ·  Cache-Control: no-store
  EV-->>RD: XADD events:stream (không chặn response)
```

**Điểm cần nhớ:** `parseUA` chạy **đúng một lần** (trong `buildClickEvent`), và nó chạy **sau**
khi đã quyết định `Location`. Toàn bộ chi phí thêm của trụ Measure/Optimize/Govern trên một link
tắt hết mọi tính năng là **0 lượt I/O**.

## D2 — Click có routing + A/B + conversion tracking

```mermaid
sequenceDiagram
  autonumber
  actor V as Người xem
  participant R as app/[key]/route.ts
  participant CT as lib/routing buildContext
  participant RT as lib/routing/resolve.ts
  participant AB as lib/ab.ts
  participant NA as resolveNavigation
  participant M as lib/measure/click-id.ts
  participant EV as lib/events.ts

  V->>R: GET /{key}
  R->>R: resolveLink + guard (như D1)
  R->>M: generateClickId() — CHỈ khi trackConversion
  R->>CT: buildContext(req) — geo/device/lang/giờ từ header
  R->>RT: resolveRouting(routing, ctx, link.url, bucketClosure)
  alt một rule match
    RT-->>R: dest, variant = ""
    Note over R,AB: KHÔNG Set-Cookie — request match rule<br/>không phát sinh định danh mới
  else không rule nào match, có abTest
    RT->>AB: bucketClosure(variants)
    AB->>AB: readVisitorId(req) ?? newVisitorId()
    AB->>AB: FNV-1a(visitorId:linkId) → cộng dồn weight
    AB-->>RT: variant
    RT-->>R: dest, variant — kèm setCookie (lazy)
  end
  R->>NA: resolveNavigation(dest, device, routing, link.url)
  Note over NA: RE-VALIDATE scheme TẠI navigate<br/>— không tin đích đã lưu
  alt app-scheme + mobile
    NA-->>R: mode = interstitial
    R-->>V: 200 HTML deferred deep link (đã escape)
  else http/https
    NA-->>R: url
    R->>M: appendClickId(url, cid) — strip cid cũ + canonical
    R->>EV: buildClickEvent(link, req, variant, cid, chainVisitorId)
    R-->>V: 302 Location · Set-Cookie: 9lv (chỉ khi bucket) · no-store
  end
```

**Hai ràng buộc không hiển nhiên:**

1. **Cookie `9lv` chỉ sinh khi thật sự bucket.** Lazy qua closure side-effect. Nếu sinh cho mọi
   click thì ADR 0046 ("không thu thập gì mới") thành lời hứa sai.
2. **Không append `cid` vào app-scheme** — deep link không mang param web; click event vẫn có cid.
   Và **không forward query của request xuống đích** ⇒ marker `?qr=1` không rò.

## D3 — Link có mật khẩu

```mermaid
sequenceDiagram
  autonumber
  actor V as Người xem
  participant R as app/[key]/route.ts
  participant PG_ as lib/govern/password-gate.ts
  participant PG as Postgres

  V->>R: GET /{key}
  R->>R: resolveLink + isLinkRedirectable
  R->>PG_: hasValidPasswordCookie(req, link.id)
  alt cookie 9lp_{id} hợp lệ (HMAC + chưa hết hạn)
    PG_-->>R: true → đi tiếp D1/D2/D4/D5
  else chưa có cookie
    PG_-->>R: false
    R-->>V: 200 HTML form mật khẩu · no-store
    Note over R,V: KHÔNG 404 (không rò tồn tại) · KHÔNG 302 (chưa lộ đích)<br/>KHÔNG đếm click (lượt này không phục vụ)
  end

  V->>R: POST /{key}  body: password=...
  R->>R: isLinkRedirectable — CÙNG guard với GET
  R->>R: readFormDataCapped(req, 16KB) — TRẦN BYTE TRƯỚC KHI PARSE
  alt vượt trần
    R-->>V: 413 text trần (không trả form-báo-sai: 413 ≠ "mật khẩu sai")
  end
  R->>PG_: verifyLinkPassword(password, passwordHash)
  Note over PG_: PBKDF2 constant-time. KHÔNG log plaintext
  alt sai
    R-->>V: 200 form lại, kèm error (không 302, không lộ đích)
  else đúng
    R->>PG_: signPasswordCookie(link.id)
    R-->>V: 302 về CHÍNH path · Set-Cookie: 9lp_{id}
  end
```

**Vì sao POST phải lặp lại guard:** xem §1.1 của Level 3 — M24 làm POST **sinh ra tiền**, và một
nhánh POST không guard là cửa sau đi vòng qua toàn bộ trụ Govern.

## D4 — Qua cổng quảng cáo: hai chặng, một loại tiền tự đo được

```mermaid
sequenceDiagram
  autonumber
  actor V as Người xem
  participant R as app/[key]/route.ts
  participant GS as adgate/serve.ts
  participant TK as adgate/token.ts + captcha.ts
  participant FR as adgate/fraud.ts
  participant NET as AdNetwork (ổ cắm ⑤)
  participant IM as adgate/impression.ts
  participant CH as ClickHouse events
  participant ST as adgate/settle.ts (worker)
  participant PG as Postgres ad_earnings

  V->>R: GET /{key}
  R->>R: sau password, TRƯỚC click-limit, và KHÔNG chạy khi pageId != null
  R->>GS: adGateGetResponse(link, req, path)
  GS->>GS: hasValidGatePass(cookie 9lg_{id}) ?
  alt đã có vé 60s
    GS-->>R: null
    R-->>V: 302 chuẩn — ĐUÔI 302 KHÔNG ĐỔI MỘT DÒNG
  else chưa có vé
    GS->>TK: signAdGateToken(9la:1:linkId:stage:iat:nonce)
    GS->>NET: slotHtml(ctx) — ĐỒNG BỘ, zero I/O
    GS->>GS: adGateCsp() + nonce riêng cho trang cổng
    GS-->>V: 200 HTML chặng 1 · nút "Tiếp tục" LUÔN CÓ TRONG DOM
    Note over V,GS: JS chỉ vô hiệu hoá nút tới hết giờ.<br/>Tắt JS vẫn dùng được và vẫn không rút ngắn được một giây
  end

  V->>R: POST /{key}  gate_token + gate_answer
  R->>GS: adGatePostResponse(link, req, path)
  GS->>TK: verifyAdGateToken — kiểm linkId · stage · iat · nonce
  GS->>TK: verifyCaptcha — tái tạo đáp án từ nonce bằng HMAC, KHÔNG lưu trạng thái
  GS->>FR: consumeNonce + evaluateImpression
  FR-->>GS: verdict { countable, reason }
  alt còn chặng nữa
    GS-->>V: 200 HTML chặng kế
  else hết chặng
    GS->>IM: recordImpression(verdict) — trong after()
    IM-->>CH: XADD event_type=impression, event_name=counted|bot|replay|duplicate|velocity|unverifiable, value=0
    GS-->>V: 302 về chính path · Set-Cookie: 9lg_{id} (vé 60s)
  end

  Note over ST,PG: SAU ĐÓ, theo nhịp worker — KHÔNG trên đường phục vụ
  ST->>CH: SELECT count lượt counted theo (ngày, nước)
  ST->>NET: refreshRates() rồi rates() — ÉP LÀM MỚI trước khi kết sổ
  ST->>ST: floor(impressions × cpmMinor / 1000), minor units
  ST->>PG: UPSERT ad_earnings theo khoá tự nhiên 5 cột, status=pending, availableAt=+holdDays
```

**Nguyên tắc đứng trên tất cả — hai hệ quả tách rời (ADR 0043 §4):**

| | |
|---|---|
| ① người xem có được phục vụ không | **LUÔN LUÔN CÓ**, không ngoại lệ. Không lối ra nào của `serve.ts` là 4xx/5xx |
| ② chủ link có được tính tiền không | chỉ khi **mọi** vế kiểm được **và** sạch |

Gộp lại là cách mọi nền tảng paid-to-click tự bắn vào chân mình: một dương-tính-giả biến thành
"link chết" với người thật — mà người thật **không khiếu nại**, họ chỉ không quay lại.

Hệ quả thứ hai: **"không kiểm được" ≠ "sạch"**. Redis chết, thiếu khoá HMAC, cấu hình lệch ⇒
`unverifiable` ⇒ không tính tiền, còn visitor vẫn đi tiếp.

## D5 — Link gắn bio page: cùng một URL, hai hành vi

```mermaid
sequenceDiagram
  autonumber
  actor V as Người xem
  participant R as app/[key]/route.ts
  participant PS as publish/serve.ts
  participant PC as publish/page-cache.ts
  participant RD as Redis
  participant PR as publish/render.ts
  participant SLOT as adgate/page-slot.ts
  participant RAIL as monetize/rail.ts
  participant EV as lib/events.ts

  V->>R: GET /{key}   (pageId != null)
  R->>R: MỌI guard Govern chạy TRƯỚC (flagged/expiry/password/click-limit)
  R->>PS: resolvePageForLink(link)
  PS->>PC: page cache (blockLinks nhúng sẵn ⇒ view cache-hit ZERO query PG)
  PS->>PS: shouldServePage(page, link)
  alt page null / unpublished / flagged / khác workspace
    PS-->>R: false
    R-->>V: fall-through nhánh 302 — Y NHƯ KHÔNG CÓ PAGE
  else phục vụ được
    PS-->>R: page + linkHrefs
    R->>SLOT: pageAdSlot({page, req, linkId}) — ĐỒNG BỘ, zero I/O (kho nằm trong RAM)
    R->>RAIL: getPaymentRail() != null — CHỈ đọc env
    R->>PR: renderPageHtml(page, {showBadge, linkHrefs, railAvailable, adSlotHtml, adSlotLabel})
    Note over PR: Renderer THUẦN — không tự đọc env (bất biến M8/T8.3)
    PR-->>R: HTML tự-chứa, escape tuyệt đối
    R->>EV: buildViewEvent — after()
    R-->>V: 200 HTML · CSP khoá chặt<br/>default-src 'none' · img-src https · style-src 'unsafe-inline'<br/>frame-ancestors 'none' · base-uri 'none' · form-action 'none'
    opt có ô quảng cáo
      R->>EV: schedulePageAdView — Event thứ hai (impression/counted_page) + lượt tính tiền advertiser, TOÀN BỘ I/O trong after()
    end
  end
```

CSP nguyên văn của trang bio (ghi lại ở đây vì sơ đồ không chứa được dấu `;`):

```
Content-Security-Policy:
  default-src 'none'; img-src https:; style-src 'unsafe-inline';
  frame-ancestors 'none'; base-uri 'none'; form-action 'none'
```

`frame-ancestors`/`base-uri`/`form-action` **không fallback** về `default-src` — khai tường minh để
chống clickjacking/overlay-phishing lên một trang bio công khai (điều kiện security-review T7.3).
Kèm `X-Content-Type-Options: nosniff` và `Cache-Control: no-store`.

**Ba điều kiện khiến nhánh này an toàn để bật trên đường nóng:**

1. `pageId == null` ⇒ short-circuit qua một phép so falsy ⇒ 302 **byte-identical** mốc trước.
2. Renderer **thuần** ⇒ CSP của bio page không đổi một byte khi bật monetize hoặc ô quảng cáo
   (có test gác).
3. Lỗi PG/chậm giữa lúc render ⇒ **fail-soft** về nhánh 302, không 500 trang public.

## D6 — Ingest Event: từ `after()` tới đúng cụm ClickHouse

```mermaid
sequenceDiagram
  autonumber
  participant R as route handler (web)
  participant ER as lib/events-route.ts
  participant RES as lib/govern/residency.ts (ổ cắm ④)
  participant RD as Redis Stream
  participant W as worker loop ①
  participant ING as lib/events-ingest.ts
  participant POOL as lib/clickhouse-pool.ts
  participant CH as ClickHouse cụm N

  R->>ER: resolveEventTarget(workspaceId, region, ts)
  ER->>RES: clusterForWorkspaceCached()
  alt MỘT cụm (OSS / self-host / prod hôm nay)
    RES-->>ER: "default" → stream events:stream (tên CŨ, giữ nguyên)
  else đa cụm
    RES-->>ER: clusterId
    ER-->>RD: XADD events:stream:{clusterId} MAXLEN ~ N
  end
  Note over ER,RD: FAIL-CLOSED: không phân giải được cụm ⇒<br/>XADD events:stream:unrouted (có MAXLEN) ⇒ TỪ CHỐI ghi,<br/>KHÔNG BAO GIỜ rơi về cụm mặc định

  W->>RD: XREADGROUP GROUP ch-writers worker-{pid} COUNT n BLOCK ms (kết nối RIÊNG)
  W->>ING: ingestEventEntries(entries)
  alt payload hỏng (lỗi DỮ LIỆU)
    ING-->>RD: XADD events:dead  +  XACK entry đó
    Note over ING: MỘT event hỏng không giết 200 event khoẻ
  else cụm chưa cấu hình / CH chết (lỗi HẠ TẦNG)
    ING-->>W: ném — KHÔNG dead-letter, KHÔNG rơi về cụm mặc định
    Note over W: Cả lô GIỮ NGUYÊN TRONG PEL
  else lành
    ING->>POOL: getClickhouse(clusterId) — gọi BÊN TRONG hàm async, CỐ Ý
    POOL->>CH: INSERT JSONEachRow (map theo TÊN cột ⇒ row cũ thiếu cột additive nhận DEFAULT)
    ING-->>RD: XACK
  end

  W->>RD: XLEN events:stream:unrouted
  Note over W: CHỈ XLEN. TUYỆT ĐỐI KHÔNG XGROUP CREATE/XREADGROUP trên stream này —<br/>ngày có một consumer insert từ đây vào một cụm là ngày fail-closed thành fail-open
```

**Đường đọc đối xứng:** `lib/analytics-fanout.ts` fan-out theo `WorkspaceRegionSpan` (bản đồ
**thời gian → cụm**) rồi gộp, và trả kèm `residencyCaveats` — dashboard **nói ra** khi một con số
không phủ trọn lịch sử, thay vì im lặng cộng hai cụm.

## D7 — Attribution: từ cú click tới một chuyển đổi không bịa được

```mermaid
sequenceDiagram
  autonumber
  actor D as Trang đích
  participant CV as app/api/track/convert
  participant RL as govern/rate-limit.ts
  participant CF as measure/convert-fraud.ts
  participant RD as Redis
  participant CW as measure/cid-workspace.ts
  participant EV as lib/events.ts
  participant CH as ClickHouse

  Note over D: trang đích đọc 9l_cid từ query
  D->>CV: POST /api/track/convert { cid, event?, value?, idempotencyKey? }<br/>hoặc GET /api/track/convert.gif?cid=...
  CV->>CV: Zod .strict() — cid phải khớp /^[0-9a-f]{32}$/
  alt request VÔ DANH mang idempotencyKey
    CV-->>D: 400 — chấp nhận nó ở đó là trao cho kẻ tấn công<br/>quyền tự cấp khoá chống-trùng của chính hắn
  end
  CV->>RL: rateLimit(RL_CONVERT)
  CV->>CW: workspace của cid (bản đồ cid→cụm, TTL 30 ngày — ADR 0038)
  CV->>CF: judgeConvert(cid, goal, value, idem)
  CF->>RD: dedup cửa sổ 24h · cap 6 goal/cid · velocity workspace · ngân sách ngày
  alt mọi vế kiểm được và sạch
    CF-->>CV: outcome = "counted"
    CV->>EV: buildConvertEvent
    EV-->>CH: XADD event_type=convert, event_name=goal, value, click_id=cid
    CV-->>D: { ok: true }  /  pixel 1×1
  else replay · over_budget · ws_rate
    CF-->>CV: outcome = lý do
    CV->>EV: vẫn ghi Event kèm KẾT CỤC
    CV-->>D: phản hồi KHÔNG PHÂN BIỆT ĐƯỢC với một cid chưa từng click
  else unverifiable (Redis chết, thiếu khoá)
    CF-->>CV: "unverifiable" — KHÔNG BỊA ra counted
    CV-->>D: như trên
  end
```

**Bất biến được đo bằng `GATE-ORACLE`:** bão hoà trần qua link A ⇒ phản hồi cho link B **cùng
workspace** phải không phân biệt được với một cid chưa từng click. Nếu phân biệt được, trần chống
gian lận trở thành một **oracle** cho kẻ tấn công dò ranh giới tenant.

**Multi-touch (ADR 0046):** `chain_key = SHA-256(9lv | tuần-ISO | salt)` cắt 128 bit — nối các
điểm chạm thành chuỗi mà **không thu thập gì mới**. Độ phủ là một **con số phải đo** (`/measure`),
không phải một giả định: chuỗi chỉ hình thành cho visitor đã có `9lv`, tức đã từng chạm một link
có A/B.

## D8 — Tiền: từ sổ cái tới ví tới một lượt rút

```mermaid
sequenceDiagram
  autonumber
  participant ST as worker: settleAdEarnings
  participant PG as Postgres
  actor P as Chủ link
  participant SU as payout-stepup.ts
  participant PO as adgate/payout.ts
  participant CH_ as adgate/wallet.ts
  actor OP as Operator
  participant MAIL as earning-notify.ts

  loop theo nhịp, có throttle và cửa sổ chu kỳ
    ST->>PG: UPSERT ad_earnings (khoá tự nhiên 5 cột) status=pending, availableAt=+holdDays
    ST->>PG: thăng hạng pending → available theo availableAt
  end

  ST->>PG: ripeEarningWorkspaces() — đọc ứng viên TRƯỚC lượt thăng hạng
  ST->>MAIL: notifyEarningMilestones()
  alt currentRates() == null
    MAIL-->>PG: skipped: "no-rates" — KHÔNG gửi cho ai
  else có bảng giá
    MAIL->>PG: CAS trên firstEarningMailedAt / payoutReadyMailedAt (fail-soft TỪNG workspace)
  end

  P->>CH_: xem ví: available · pending · reversed · paid, theo link và theo nước
  Note over CH_: tiền quy về LINK bằng PHÂN RÃ largest-remainder,<br/>KHÔNG bằng cột linkId trong bảng tiền (DECISIONS 2026-08-24)
  P->>SU: yêu cầu rút
  SU->>SU: phiên phải trẻ hơn payoutStepUpMaxAgeSec() (mặc định 900s)
  Note over SU: 900s là con số BẢO MẬT, không phải tiện lợi:<br/>nới nó là nới đúng cửa sổ một cookie bị chiếm dùng được<br/>ở bề mặt DUY NHẤT trong repo chuyển tiền ra khỏi hệ
  SU->>PO: requestPayout(workspaceId, method)
  PO->>PG: kiểm MỘT yêu cầu đang mở (partial unique index)
  PO->>PG: CREATE payout_requests status=requested
  PO->>PG: UPDATE ad_earnings SET status=claimed, payoutRequestId=...<br/>WHERE status=available — ĐẾM ROWCOUNT
  Note over PO,PG: Hàng rào chống CHI TRÙNG là CAS trên HÀNG WORKFLOW,<br/>không phải đọc số dư rồi tin nó còn đúng lúc ghi
  alt không claim được dòng nào
    PO->>PG: status=rejected, note=nothing-to-claim
  else dưới ngưỡng
    PO->>PG: status=rejected, note=below-threshold
  end

  OP->>PG: duyệt tại /govern/payouts hoặc pnpm ops:payout
  OP->>PO: markPayoutPaid(requestId)
  PO->>PG: ad_earnings claimed → paid · payout_requests → paid
  PO->>PG: reset cột "đủ ngưỡng" để mail mốc sau lại bắn
  Note over OP: v1 dừng ở approved/paid — CHUYỂN TIỀN DIỄN RA NGOÀI HỆ, có chủ ý<br/>(KYC/thuế/W-9/1099 là [human], không phải nợ mã)
  alt từ chối
    OP->>PO: rejectPayout(requestId, note)
    PO->>PG: releasePayoutClaim — WHERE status=claimed ⇒ dòng ĐÃ paid không bị kéo ngược
  end
```

**Quarantine cắt vào đâu:** `quarantineLink` thu hồi thu nhập **đúng một lần** tại thời điểm chặn,
theo **hai mức tuỳ ai ra phán quyết**:

| Ai | Thu hồi | Vì sao |
|---|---|---|
| **operator** | tất cả, kể cả phần đã mở khoá | Nếu không, kẻ farm chỉ cần chờ hết hạn giữ |
| **quét tự động** | chỉ đóng băng `pending` | Dương-tính-giả không được cướp tiền người vô tội |

Và `reversed` **thôi là trạng thái chết, nhưng chỉ theo một chiều**: phán quyết của **máy**
(`reversedSource='scan'`) đảo ngược được khi quarantine được gỡ; của **người** thì không. `report`
(báo cáo vô danh) xếp vào MÁY — một biểu mẫu công khai không được có quyền xoá vĩnh viễn thu nhập
của người khác.

## D9 — Stripe billing webhook: single-writer của tiền

```mermaid
sequenceDiagram
  autonumber
  participant S as Stripe
  participant WH as app/api/stripe/webhook
  participant RAIL as PaymentRail (ổ cắm ①)
  participant IDEM as BillingWebhookEvent
  participant PG as Postgres workspaces
  participant EV as lib/events.ts
  participant EM as connect/emit.ts

  S->>WH: POST (RAW body + header chữ ký)
  WH->>RAIL: verifyWebhook(rawBody, signatureHeader)
  alt secret thiếu
    RAIL-->>WH: THROW → 5xx (lỗi CẤU HÌNH, không phải 400)
  else chữ ký sai / hết hạn / body hỏng
    RAIL-->>WH: null → 400
  else hợp lệ
    RAIL-->>WH: StripeWebhookEvent
  end
  WH->>IDEM: ghi event id — replay ⇒ bỏ qua, không ghi hai lần
  WH->>PG: UPDATE workspace: plan, currentPeriodStart (MỐC NEO), currentPeriodEnd
  Note over WH,PG: GHI DUY NHẤT TỪ WEBHOOK ĐÃ VERIFY (ADR 0017 §3).<br/>Xoá khi customer.subscription.deleted
  opt event là một sale được thanh toán
    WH->>PG: MonetizeSale paid
    WH->>EV: enqueueEventStrict(earn) — THROW nếu không ghi được
    WH->>EM: emitWebhookEvents("sale.paid") — BEST-EFFORT, nuốt lỗi
  end
  opt /ee chiếm ổ ②
    WH->>RAIL: handler đã đăng ký (vd account.updated)
  end
  WH-->>S: 200
```

**Vì sao `currentPeriodStart` là mốc neo chứ không phải "đầu cửa sổ đang đo":** cửa sổ đo lặp
**hằng tháng** từ mốc neo, vì `PLAN_MONTHLY_SERVE_ALLOWANCE` là trần **tháng** — còn gói **năm**
có `currentPeriodEnd` cách 12 tháng. Lấy thẳng chu kỳ Stripe làm cửa sổ = cho khách gói năm
**12× trần** trong một kỳ (ADR 0042).

`currentPeriodEnd` **không tham gia tính cửa sổ**, cố ý: webhook trễ/mất một kỳ thì cửa sổ vẫn
phải tự trôi, không được đứng im khoá khách trong kỳ cũ.

## D10 — Webhook OUT: tín hiệu, không phải Event

```mermaid
sequenceDiagram
  autonumber
  participant W as write path (createLink / publish / Stripe)
  participant EM as connect/emit.ts
  participant PG as Postgres webhook_endpoints
  participant RD as Redis
  participant WL as worker loop ②
  participant DL as connect/deliver.ts
  participant EP as Endpoint khách

  W->>EM: emitWebhookEvents(type, data) — SAU khi sự thật đã ghi
  EM->>PG: SELECT endpoint enabled + subscribe đúng type
  alt không có endpoint nào
    EM-->>W: return sớm — ZERO XADD
  else có
    EM->>RD: XADD webhooks:deliveries (stream RIÊNG, không đụng events:stream)
    Note over EM: BEST-EFFORT: mọi lỗi được NUỐT.<br/>KHÔNG BAO GIỜ reject, không làm webhook Stripe trả 5xx khi Redis chết
  end

  WL->>RD: XREADGROUP (kết nối BLOCK riêng) + XAUTOCLAIM orphan theo minIdle
  WL->>DL: handleDelivery(msg)
  DL->>DL: SSRF-check: https công khai, từ chối IP nội bộ/loopback/link-local,<br/>hostname không dấu chấm, URL kèm userinfo
  DL->>EP: POST JSON + header 9link-signature (HMAC-SHA256, secret whsec9_)
  alt 2xx
    DL->>RD: XACK
  else thất bại / timeout
    DL->>RD: ZADD webhooks:retry (điểm = thời hạn)
    DL->>PG: đếm failure, quá ngưỡng ⇒ TỰ TẮT endpoint
  end
  WL->>RD: requeueDueRetries() mỗi vòng
  WL->>RD: cleanupLeakedConsumers()
```

**Payload không bao giờ chứa PII** — chỉ `id` + field nghiệp vụ. Và `link.deleted` **không kèm
`url`**: gửi kèm đích của một link vừa bị xoá là phát tán thêm một bản dữ liệu của khách mà không
ai cần.

**Loại mới không tự chảy vào endpoint cũ.** Endpoint chỉ nhận đúng những type nó đã đăng ký. 9Link
cố ý **không** tự thêm hộ — vì đó vừa là đổi hợp đồng sau lưng consumer, vừa là một cú bùng nổ lưu
lượng (`bulk` 100 link × N endpoint).

## D11 — Chống lạm dụng: bốn lớp, và chỗ mỗi lớp đứng

```mermaid
sequenceDiagram
  autonumber
  actor U as Chủ link
  participant API as write path
  participant US as govern/url-safety.ts
  participant BL as govern/blocklist.ts
  participant GSB as Google Safe Browsing
  participant PG as Postgres
  participant WL as worker loop ③
  actor A as Người báo cáo (vô danh)
  actor OP as Operator

  U->>API: tạo/sửa link (mọi đích: url + routing dests + page blocks)
  API->>US: checkUrlSafety(url, lane="write")
  US->>BL: lookupBlocked(canonicalizeUrl(url))
  Note over BL: chuẩn hoá URL: một DẤU CHẤM CUỐI từng né được cả blocklist<br/>(evil.example.) — lỗ THẬT đã vá ở M18
  alt trúng mục OPERATOR thêm
    BL-->>API: hardReject → 422 (cửa hẹp cố ý)
  else trúng seed / feed / GSB
    US->>PG: flagged = true
    Note over API,PG: edge REFUSE — ADR 0012. Không quét live ở hot path
  else GSB lỗi / hết ngân sách
    US-->>API: degraded = true = "KHÔNG BIẾT" ≠ "sạch"
    Note over API,PG: KHÔNG gỡ cờ flagged cũ vì một lần "không biết"
  end

  loop rescanLoop, nhịp poll, throttle
    WL->>PG: SELECT link WHERE nextScanAt <= now (index riêng)
    WL->>GSB: quét lại, làn ngân sách "job"
    Note over WL: link ĐANG KIẾM TIỀN ⇒ chu kỳ dày gấp 4 (24h → 6h).<br/>Link kiếm tiền có ĐỘNG CƠ TÀI CHÍNH để lật nội dung sau khi đã qua cửa quét
    alt sạch N lần liên tiếp + source=scan + chưa từng bị operator upheld
      WL->>PG: TỰ LÀNH — gỡ quarantine
      Note over WL,PG: thiếu vế tự lành thì một dương-tính-giả =<br/>link chết IM LẶNG VĨNH VIỄN
    else không an toàn
      WL->>PG: quarantineLink(source="scan") — DÍNH, đường ghi người dùng không bao giờ chạm
      WL->>PG: thu hồi thu nhập CHỈ phần pending
    end
  end

  A->>API: POST /api/report (không tài khoản, cap per-IP + cap instance, IP đã HMAC)
  API->>PG: tạo AbuseCase
  Note over API,PG: báo cáo KHÔNG BAO GIỜ tự chặn link nào —<br/>nếu có thì bất kỳ ai cũng hạ được link bất kỳ với chi phí 0

  OP->>PG: /govern/abuse — giữ chặn / chặn cả host / gỡ chặn
  alt operator gỡ chặn một link đang kiếm tiền
    OP->>PG: quarantine(source="operator") ⇒ thu hồi TẤT CẢ, kể cả phần đã mở khoá
  end
  OP->>BL: pnpm blocklist — thêm/bớt mục chặn TOÀN CỤC, KHÔNG CẦN DEPLOY (ADR 0034)
```

## D12 — Boot: thứ tự là một phần của phép đo

```mermaid
sequenceDiagram
  autonumber
  participant NX as Next.js
  participant INS as instrumentation.ts
  participant EE as ee/entry.ts (hoặc lib/ee-entry.ts rỗng)
  participant S as 5 registry
  participant RP as residency-install.ts
  participant PG as Postgres (mọi cụm)

  NX->>INS: register() — ĐÚNG MỘT LẦN mỗi tiến trình, trước request đầu
  INS->>INS: if NEXT_RUNTIME !== "nodejs" return
  INS->>EE: import("@/lib/ee-entry") → loadEeExtensions()
  Note over INS,EE: next.config.mjs NormalModuleReplacementPlugin<br/>thay module khi ee/entry.ts CÓ TRÊN ĐĨA

  EE->>EE: if eeSsoConfigured() → createSamlSsoProvider()
  EE->>S: ③ registerSsoProvider · finally sealSsoRegistry()
  Note over EE: cấu hình NỬA VỜI ⇒ NÉM ngay đây, không phải ở lần đăng nhập đầu của người thật

  EE->>EE: if eeResidencyConfigured() → createEeResidencyRouter()
  EE->>S: ④ registerResidencyRouter · finally sealResidencyRegistry()
  Note over EE: PHẢI ĐỨNG TRƯỚC khối rail — khối rail có return sớm

  EE->>EE: if eeAdNetworkConfigured() → eeAdRates() rồi registerAdNetwork()
  EE->>EE: installAdChargeShutdownFlush() — SIGTERM flush đệm tính tiền advertiser
  EE->>S: ⑤ seal ở MỌI nhánh
  Note over EE: chiếm được ổ này là chèn được mã tuỳ ý lên MỌI cổng đang sống,<br/>cho MỌI visitor vô danh, mà không một cổng nào đỏ

  EE->>EE: if !eeRailConfigured() return
  EE->>EE: takeRateBps() — đọc NGAY LÚC BOOT dù chưa dùng
  EE->>S: ① registerRail · ② registerStripeEventHandler
  EE->>S: finally seal cả hai ổ

  INS->>RP: bootResidencyPostgres("server")
  RP->>RP: gác cấu hình PII → cắm định tuyến → SEAL trong finally
  INS->>PG: forEachCluster(assertOperatorAllowlistSane)
  Note over INS,PG: email allowlist MỒ CÔI là lỗ hòm thư — prod TỪ CHỐI BOOT.<br/>Chạy TRÊN TỪNG CỤM: một operator mồ côi ở cụm UAE vẫn là cửa hậu cấp instance
  INS->>PG: forEachCluster(assertResidencyRoutingSane)
  Note over INS,PG: thiếu EE_RESIDENCY_CLUSTERS ⇒ server vẫn 200, đường ghi lặng lẽ<br/>quay về MỘT cụm, Event của workspace UAE chảy vào cụm mặc định
```

**Vì sao thứ tự ở khối này là một phần của phép đo (M30/T30.6):** bản đầu đặt việc cắm ổ PII
**sau** hai phép gác đọc DB. Lúc hai phép gác đó chạy, resolver còn `null` ⇒ `prisma` trỏ cụm mặc
định ⇒ chúng chỉ còn đo **1/N vùng**. Hai kiểm soát an ninh có từ M18/M19 bị **hạ cấp im lặng** —
không cổng nào đỏ, không log nào đổi.

**Worker cũng nạp ổ cắm** (`loadSeams()` trong `worker/index.ts`): không có nó thì job kết sổ
không biết giá, và nó sẽ im lặng kết sổ với `rates() == null` — tức không một dòng sổ nào.
