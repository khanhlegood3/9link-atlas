# Atlas 9Link 5W1H — bản công khai

Một trang tài liệu tương tác, **một file HTML tự chứa**: [`index.html`](index.html) · xem tại
**https://khanhlegood3.github.io/9link-atlas/** sau khi GitHub Pages được bật.

Kèm theo là **bộ mô hình C4** của cùng hệ thống đó, ở [`c4/`](c4/) — bảy file Markdown có sơ đồ
Mermaid. Đọc C4 qua **blob view của GitHub** ([`c4/`](c4/) → mở từng file) thì sơ đồ được render;
qua Pages thì chỉ có markdown thô.

Trang trình bày lại các tuyên bố trong white paper của **9Link** — một nền tảng "All in One for
Link" mã nguồn mở — dưới dạng 6 thẻ 5W1H → 5 phần → 18 chiều đo → mô hình kinh tế quảng cáo có
thanh trượt → quiz 12 câu, kèm **nhãn độ tin cậy** cho từng con số (`Đã đo` · `Đã viết, chưa chạy`
· `Mô hình` · `Chưa đo được`) và một **log sửa lỗi công khai** trong đó tài liệu tự nêu những lần
nó tự khai lệch số đo thật.

## Repo này KHÔNG chứa mã nguồn của 9Link

9Link là một repo **private**. Repo này chứa **tài liệu** đã được xuất bản từ đó — một trang HTML và
một bộ mô hình kiến trúc — cộng vài file để chúng tự phục vụ được. Không có mã sản phẩm, không có
schema, không có migration, không có hạ tầng, không có bí mật.

Toàn bộ những gì ở đây:

| File | Là gì | Cổng đã kiểm nó ở repo gốc |
|---|---|---|
| `index.html` | trang Atlas — một file HTML tự chứa, CSS/JS nội tuyến, 0 tài nguyên ngoài. Số byte và md5 xem `SOURCE.json` | `scripts/check-atlas.mjs` |
| `c4/*.md` | bộ mô hình C4 (7 file): System Context · Container · Component · Code · Dynamic views · Deployment · README | `scripts/check-vn-docs.mjs` |
| `SOURCE.json` | **nguồn gốc của đúng bytes đang nằm ở đây** — đọc mục dưới | — |
| `README.md` | file này | — |
| `.github/workflows/deploy.yml` | đăng lên GitHub Pages, sau khi kiểm toàn vẹn cả trang lẫn bộ C4 | — |
| `.gitignore` | bỏ qua `_site/` (thư mục artifact của workflow) | — |

### ⚠️ Bộ C4 là một ẢNH CHỤP CÓ NGÀY, không phải hiện trạng

Bảy file trong `c4/` được viết ngược từ mã nguồn thật tại commit **`b559c68`** (2026-09-08), và
**tự khai điều đó ở đầu mỗi file**. Repo gốc đã đi tiếp từ ngày ấy. Ví dụ đã đo bằng `git ls-tree`:
`docs/adr/` có **47** file `.md` tại `b559c68` và **50** tại `4af8932` (2026-11-10) — cả hai đều
đúng, tại cây của mình. Đọc các con số trong `c4/` như một tài liệu kiến trúc **có ngày**, và nếu
cần hiện trạng thì đo lại trên cây đang đứng chứ đừng tin một con số đã hai tháng tuổi.

Trang `index.html` thì khác: nó ghim cây số liệu của riêng nó, và khai cây đó ngay trong thân trang
(*"Trang này đo tại `SHA`"*). Hai tài liệu, hai ngày đo — và cả hai đều tự khai, đó là điểm chung.

## `SOURCE.json` — vì sao nó tồn tại

Trang này là **bản sao**, và một bản sao không tự nói được nó sao từ đâu, lúc nào, và bản gốc có
được kiểm hay không. `SOURCE.json` trả lời ba câu đó:

```json
{
  "source_repo":    "khanhlegood3/9Link",
  "source_sha":     "SHA của commit mà file trang và bộ C4 được lấy ra",
  "source_path":    "docs/atlas/index.html",
  "published_at":   "thời điểm xuất bản",
  "page_md5":       "md5 của đúng bytes index.html trong repo này",
  "page_bytes":     "<số byte của đúng file index.html trong repo này>",
  "c4_source_path": "docs/c4",
  "c4_dir":         "c4",
  "c4_files":       "<số file .md trong c4/>",
  "c4_bytes":       "<tổng số byte của chúng>",
  "c4_manifest":    "01-system-context.md=<md5> 02-container.md=<md5> … — MỘT dòng, mỗi cặp tên=md5",
  "content_md5":    "danh tính của CẢ lần xuất bản: md5 của page_md5 + c4_manifest",
  "data_sha":       "SHA của cây mà SỐ LIỆU trong trang được đo — KHÁC source_sha",
  "gate":           "các cổng đã chạy và thoát 0 TRƯỚC khi file này được ghi"
}
```

Hai khoá cần giải thích vì chúng dễ nhầm:

- **`content_md5`** là khoá mà phép *idempotent* khoá theo: chạy lại script trên cùng một nội dung thì
  con tem **không bị ghi lại**, nên `published_at` nghĩa là *lần đầu tiên đúng nội dung này được xuất
  bản*. Nó phải gộp cả `c4_manifest` chứ không chỉ `page_md5` — nếu chỉ khoá theo trang thì "sửa C4
  mà không sửa trang" sẽ được báo là *nội dung không đổi*, và một con tem không hề khai C4 được giữ
  nguyên trong khi cây ở đây đã đổi.
- **`c4_manifest`** cố tình là MỘT dòng gồm các cặp `tên=md5`, không phải một mảng JSON: workflow ở
  repo này đọc `SOURCE.json` bằng `grep` + vòng lặp bash chứ không bằng `jq`, và định dạng được giữ
  cố định CHÍNH XÁC để phép đọc đó đáng tin. Đổi định dạng ở đây thì phải đổi phép đọc ở đó, trong
  cùng một commit.

`scripts/publish-atlas.sh` (nằm trong repo 9Link, không nằm ở đây) **chỉ ghi `SOURCE.json` sau khi
CẢ HAI cổng thoát 0**: `check-atlas.mjs` trên trang và `check-vn-docs.mjs` trên `docs/c4/`. Vì vậy
workflow ở repo này không cần cài lại cổng nào, cũng không cần định nghĩa lại một phép kiểm nào:
nó chỉ việc so `md5(index.html)` với `page_md5`, và `md5(c4/<file>)` với từng cặp trong
`c4_manifest` — kể cả đếm ngược, để một file C4 **mồ côi** (có trên đĩa mà tem không khai) không
lọt qua. Nếu khớp thì bytes đang được đăng **là** bytes đã qua cổng — suy ra bắc cầu, một định
nghĩa, không bản sao thứ hai của định nghĩa đó.

## ⚠️ Hai SHA khác nhau, đừng gộp làm một

- `SOURCE.json.source_sha` = SHA của cây mà **file trang** được lấy ra.
- Các SHA trích dẫn **bên trong** trang (ví dụ `2d24b47`) = SHA của cây mà **số liệu trong trang**
  được đo.

Hai cây đó khác nhau và phải khác nhau: trang được đo một lần rồi có thể được sửa câu chữ nhiều
lần sau đó mà số liệu không đổi. Một con số quy mô không kèm SHA là một con số sẽ sớm sai — nhưng
SHA của *file* thì không phải SHA của *số liệu*, và nhầm hai cái là một lỗi khó thấy vì cả hai đều
trông như bằng chứng.

## Trang này nói gì về những thứ chưa chạy được

Rất nhiều phần của 9Link **bất hoạt có chủ đích** (các bảng tiền chưa có dòng nào, ba circuit
breaker đang OFF, các mốc chưa tới). Trang được viết để **không mô tả chúng như đang chạy**: mỗi
con số mang nhãn độ tin cậy, và những gì chưa đo được thì ghi là chưa đo được thay vì suy luận.
Nếu bạn đọc trang này như một cam kết về trạng thái vận hành, hãy đọc phần nhãn trước.

Trang cũng chép lại một số **khoảng trống an ninh đã ghi nhận nội bộ** (ví dụ cờ
`TRUST_EDGE_IP_HEADER`, các breaker đang OFF) vì white paper gốc ghi chúng và trang này không thêm
cũng không bớt tuyên bố nào. Việc xuất bản trang là một quyết định công bố có ý thức, không phải
tác dụng phụ của một pipeline.

Bộ `c4/` cũng vậy, và ở mức cao hơn: nó mô tả **kiến trúc** của hệ thống ở bốn mức phóng đại —
container nào nói chuyện với container nào bằng giao thức gì, component nào sở hữu quyết định gì,
volume nào gắn ở đâu. Bảy file đó đã được quét tự động trước khi công bố (token, khoá AWS/OpenAI,
JWT, private key, mật khẩu nhúng trong URL, `DATABASE_URL`, IP riêng, email, hostname nội bộ — 12
mẫu, **0 nghi vấn**), nhưng *không có bí mật* và *không tiết lộ gì* là hai chuyện khác nhau. Đây là
một quyết định công bố kiến trúc, có ý thức, chứ không phải một file tài liệu đi kèm.

## Giấy phép — CHƯA quyết định, và repo này cố tình chưa có file `LICENSE`

White paper và tài liệu của 9Link nằm dưới **AGPL-3.0** (với carve-out cho `ee/`). Trang này và bộ
`c4/` đều là tác phẩm phái sinh từ tài liệu đó. Repo này **chưa** có `LICENSE` — theo mặc định của GitHub nghĩa là
*"mọi quyền được bảo lưu"*, tức chặt hơn chứ không lỏng hơn, nên chưa có rủi ro cấp quyền nhầm.
Nhưng đó là trạng thái chờ quyết định chứ không phải đã ổn: cần chọn giữa (a) AGPL-3.0 cho nhất
quán với tài liệu gốc, (b) CC BY-SA cho tài liệu, hay (c) giữ nguyên không cấp phép công khai.

## Nếu bạn là người vận hành 9Link

Xuất bản lại sau khi sửa trang:

```bash
# trong repo 9Link
bash scripts/publish-atlas.sh --check   # chỉ kiểm: cổng xanh? trang đã commit? (không ghi gì)
bash scripts/publish-atlas.sh           # tạo repo nếu chưa có, bật Pages, push
```

Script từ chối xuất bản nếu **một trong hai cổng** đỏ, nếu thiếu `jsdom`/`mermaid` (cổng thoát
3 = *bỏ qua*, và bỏ qua thì không được coi là đạt), hoặc nếu `docs/atlas/index.html` **hay bất kỳ
file nào trong `docs/c4/`** còn thay đổi chưa commit — vì khi đó `source_sha` sẽ không trỏ tới đúng
bytes đang được đăng.
