# Mô hình C4 của 9Link

> **Tài liệu kiến trúc chuẩn C4** (Simon Brown — [c4model.com](https://c4model.com)) cho toàn bộ
> hệ thống 9Link, **viết ngược từ mã nguồn thật** tại commit `b559c68`, không viết từ ý định.
>
> Mỗi hộp trong mọi sơ đồ dưới đây đều trỏ được tới một đường dẫn trong repo. Nếu một sơ đồ nói
> rộng hơn mã, đó là **bug của tài liệu** và phải sửa tài liệu — xem §4.

---

## 0. Vì sao repo này cần một mô hình C4

`docs/ARCHITECTURE.md` có một sơ đồ 4 lớp rất tốt để **nhớ**, nhưng nó là một bức tranh tĩnh ở
một mức phóng đại duy nhất. Repo này đã đi qua **45 mốc** và **47 ADR** *(đo tại `b559c68`, 2026-09-08 — tại `4af8932` ngày 2026-11-10 là **50** ADR)*, và phần lớn những quyết
định đắt nhất nằm ở chỗ giao nhau giữa hai mức phóng đại — ví dụ:

- *"Cột này có được vào `resolveLink.select` không?"* — một câu hỏi **Level 4 (Code)** quyết định
  một lời hứa **Level 2 (Container)**: đường 302 không tốn thêm một lượt I/O nào (ADR 0003/0006/
  0008/0016/0024).
- *"Ổ cắm này ai được chiếm, và seal lúc nào?"* — một câu hỏi **Level 3 (Component)** quyết định
  một ràng buộc **pháp lý**: mã thương mại không bao giờ được phát hành dưới AGPLv3 (ADR 0029).

C4 tồn tại để trả lời đúng loại câu hỏi đó: **cùng một hệ thống, ở bốn mức phóng đại, mỗi mức một
đối tượng đọc**. Bộ tài liệu này giữ nguyên tinh thần đó.

## 1. Bốn mức + hai sơ đồ bổ sung

| # | Sơ đồ | Mức | Trả lời câu hỏi | Người đọc chính |
|---|---|---|---|---|
| 1 | [System Context](01-system-context.md) | L1 | 9Link là gì, ai dùng, nói chuyện với hệ ngoài nào | Mọi người — kể cả không kỹ thuật |
| 2 | [Container](02-container.md) | L2 | Triển khai được tạo bởi những tiến trình/kho dữ liệu nào | Kỹ sư mới · vận hành · kiến trúc sư |
| 3 | [Component](03-component.md) | L3 | Bên trong mỗi container có những khối nào, ai phụ thuộc ai | Kỹ sư làm việc trên container đó |
| 4 | [Code](04-code.md) | L4 | Hợp đồng kiểu dữ liệu & mô hình dữ liệu ở mức module/class | Kỹ sư sửa đúng module đó |
| 5 | [Dynamic views](05-dynamic-views.md) | bổ sung | 12 luồng quan trọng chạy qua các hộp trên theo thứ tự nào | Người rà soát · người vá sự cố |
| 6 | [Deployment](06-deployment.md) | bổ sung | Container nào chạy ở đâu, mạng nào, cụm nào | Vận hành · người self-host |

**Thứ tự đọc khuyến nghị:** 1 → 2 → 5 (chọn luồng bạn quan tâm) → 3 → 4.
Sơ đồ động (5) đặt trước component (3) là cố ý: ở một hệ thống mà *đường đi của một cú click* là
tài sản cốt lõi, đọc luồng trước rồi mới đọc cấu trúc khiến cấu trúc có nghĩa ngay.

## 2. Quy ước vẽ

### 2.1 Ký pháp

Sơ đồ viết bằng **Mermaid C4** (`C4Context` / `C4Container` / `C4Component` / `C4Dynamic` /
`C4Deployment`) — GitHub render được ngay trong trình duyệt, không cần công cụ ngoài. Hai sơ đồ
quan trọng nhất (L1, L2) có kèm **bản ASCII** để đọc được ở terminal, trong `git diff`, và trong
mọi nơi không render Markdown.

| Hộp Mermaid | Nghĩa trong repo này |
|---|---|
| `Person` | Một con người, **không phải** một vai trò trong RBAC. RBAC (`owner`/`admin`/`member`/`viewer`) là chi tiết Level 4 của container Dashboard. |
| `System` | Hệ thống 9Link (một bản cài — cloud hoặc self-host). |
| `System_Ext` | Hệ ngoài: ta gọi nó, nó không phải mã của ta. |
| `Container` | Tiến trình triển khai được: `web`, `worker`, `ops-cli`. |
| `ContainerDb` | Kho dữ liệu: Postgres, ClickHouse, Redis. |
| `Component` | Module/thư mục trong `apps/web/lib/**` hoặc `ee/**`. |

### 2.2 Ba luật vẽ, mỗi luật một lý do từ chính repo này

1. **Không vẽ hộp nào không trỏ được tới mã.** Mọi `Container`/`Component` đều ghi đường dẫn
   trong ô mô tả. Sơ đồ không có đường dẫn là sơ đồ về một hệ thống khác.
2. **Nét đứt = chưa có trong cây mã.** Kiến trúc đích (`docs/ARCHITECTURE.md`) nói lớp EDGE là
   Cloudflare Workers tại PoP US/UAE/EU. **Hôm nay đường 302 chạy trong Next.js route handler**
   (`apps/web/app/[key]/route.ts`) đứng sau một reverse proxy. Vẽ nó như thể đã là Workers là nói
   dối người đọc — nên nó được vẽ là một nút *đích* với nét đứt và ghi rõ ở §5 của Level 2.
3. **Ranh giới open-core luôn được vẽ.** `/ee` là một `Boundary` riêng trong mọi sơ đồ từ L2 trở
   xuống, vì chiều phụ thuộc `Core → /ee` bị **cấm bằng cổng máy** (`scripts/gate-ee-dir.mjs`) chứ
   không bằng quy ước. Một sơ đồ không vẽ ranh giới đó sẽ mời người đọc vẽ một mũi tên bị cấm.

### 2.3 Thuật ngữ nhất quán

| Thuật ngữ | Nghĩa | Nguồn |
|---|---|---|
| **trụ** (pillar) | Một trong 9 năng lực: Create · Enrich · Route · Publish · Measure · Optimize · Monetize · Govern · Connect | `CLAUDE.md §2` |
| **mặt phẳng cấu hình** | Postgres — Link, Page, Workspace, User, sổ tiền | `CLAUDE.md §1` |
| **mặt phẳng sự kiện** | ClickHouse — bảng `events` append-only, **một lớp duy nhất** | bất biến §0.2 |
| **đường nóng** / hot path | Chuỗi lệnh phục vụ `GET /{key}` trước khi trả 302 | bất biến §0.1/§0.3 |
| **ổ cắm** (socket) | 5 registry mà `/ee` chiếm lúc boot rồi **seal** | ADR 0029 |
| **cổng** (gate) | Một phép đo máy trong `scripts/verify.sh` hoặc `apps/web/e2e/*` | `CLAUDE.md §6` |
| **residency mức 1/2/3** | nhãn vùng → Event theo cụm → Postgres/PII theo cụm | ADR 0036–0039, 0047 |

## 3. Ánh xạ tài liệu ↔ mã

| Sơ đồ | Đường dẫn mã chính |
|---|---|
| L1 Context | `README.md` · `docs/PRD.md` · `.env.example` (danh sách hệ ngoài) |
| L2 Container | `apps/web/**` · `apps/web/worker/index.ts` · `apps/web/scripts/**` · `deploy/docker-compose.yml` · `packages/**` |
| L3 Component | `apps/web/lib/**` (17 thư mục) · `ee/**` (5 thư mục) |
| L4 Code | `packages/db/prisma/schema.prisma` (34 model · 55 migration) · `apps/web/lib/clickhouse.ts` (DDL) · các `interface` của 5 ổ cắm |
| Dynamic | `apps/web/app/[key]/route.ts` · `apps/web/lib/events*.ts` · `apps/web/lib/monetize/adgate/**` |
| Deployment | `deploy/docker-compose.yml` · `infra/docker-compose*.yml` · `apps/web/Dockerfile` |

## 4. Luật bảo trì (đọc trước khi sửa bất kỳ sơ đồ nào)

Repo này có một họ lỗi được ghi lại nhiều lần trong `HANDOFF.md`: **"doc nói rộng hơn code"**.
Bộ tài liệu C4 này là mục tiêu tiếp theo của họ lỗi đó, nên nó tự đặt rào:

1. **Thêm một container / kho dữ liệu mới** → sửa Level 2 **và** Level 6 trong cùng một PR.
2. **Thêm một cột vào `resolveLink.select`** (`apps/web/lib/link-cache.ts`) → sửa Level 4
   (`ResolvedLink`) **và** ghi ADR: đây là ngoại lệ hẹp với ADR 0003, và mọi ngoại lệ trước đó
   đều có một ADR riêng. Không có ADR = PR chưa xong.
3. **Thêm một ổ cắm `/ee`** → sửa Level 3 (§ ranh giới open-core) + Level 4 (interface) +
   `apps/web/lib/ee-entry.ts` (seal) **và** `ee/entry.ts` (register + seal trong `finally`).
4. **Thêm một `event_type` mới** → **không** được thêm kho dữ liệu mới (bất biến §0.2). Kiểm
   `apps/web/lib/clickhouse.ts`: nếu cần DDL mới, dừng lại và viết ADR trước.
5. **Sơ đồ nào hết đúng thì xoá, không để mập mờ.** Một sơ đồ sai đắt hơn không có sơ đồ, vì nó
   được tin.

## 5. Trạng thái tài liệu

| | |
|---|---|
| Viết từ commit | `b559c68` (`main`, 2026-09-08) |
| Ngày viết | 2026-11-08 |
| Phạm vi | Toàn bộ cây mã: `apps/web`, `packages/{db,sdk,n8n-nodes-9link}`, `ee/`, `infra/`, `deploy/`, `scripts/` |
| Quy mô đo được | ~108,7k dòng TS/TSX phi-test · ~50,7k dòng test · 240 file unit test · 78 file e2e · 46 cổng `GATE-*` trong `verify.sh` · 47 ADR · 34 Prisma model · 55 migration · 3 locale (1.683 khoá i18n) |
| Chưa phủ | UI component tree (`app/(app)/**`, `app/_home/**`) — cố ý: C4 Level 4 cho React là một bản đồ component riêng, và nó đổi nhanh hơn phần còn lại |
