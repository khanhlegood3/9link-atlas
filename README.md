# Atlas 9Link 5W1H — bản công khai

Một trang tài liệu tương tác, **một file HTML tự chứa**: [`index.html`](index.html) · xem tại
**https://khanhlegood3.github.io/9link-atlas/** sau khi GitHub Pages được bật.

Trang trình bày lại các tuyên bố trong white paper của **9Link** — một nền tảng "All in One for
Link" mã nguồn mở — dưới dạng 6 thẻ 5W1H → 5 phần → 18 chiều đo → mô hình kinh tế quảng cáo có
thanh trượt → quiz 12 câu, kèm **nhãn độ tin cậy** cho từng con số (`Đã đo` · `Đã viết, chưa chạy`
· `Mô hình` · `Chưa đo được`) và một **log sửa lỗi công khai** trong đó tài liệu tự nêu những lần
nó tự khai lệch số đo thật.

## Repo này KHÔNG chứa mã nguồn của 9Link

9Link là một repo **private**. Repo này chỉ chứa **đúng một trang** đã được xuất bản từ đó, cộng
vài file để trang tự phục vụ được. Không có mã sản phẩm, không có schema, không có hạ tầng.

Toàn bộ những gì ở đây:

| File | Là gì |
|---|---|
| `index.html` | trang Atlas — một file HTML tự chứa, CSS/JS nội tuyến, 0 tài nguyên ngoài. Số byte và md5 xem `SOURCE.json` |
| `SOURCE.json` | **nguồn gốc của đúng bytes đang nằm ở đây** — đọc mục dưới |
| `README.md` | file này |
| `.github/workflows/deploy.yml` | đăng trang lên GitHub Pages, sau khi kiểm toàn vẹn |
| `.gitignore` | bỏ qua `_site/` (thư mục artifact của workflow) |

## `SOURCE.json` — vì sao nó tồn tại

Trang này là **bản sao**, và một bản sao không tự nói được nó sao từ đâu, lúc nào, và bản gốc có
được kiểm hay không. `SOURCE.json` trả lời ba câu đó:

```json
{
  "source_repo":  "khanhlegood3/9Link",
  "source_sha":   "SHA của commit mà file trang được lấy ra",
  "source_path":  "docs/atlas/index.html",
  "published_at": "thời điểm xuất bản",
  "page_md5":     "md5 của đúng bytes index.html trong repo này",
  "page_bytes":   <số byte của đúng file index.html trong repo này>,
  "gate":         "cổng đã chạy và thoát 0 TRƯỚC khi file này được ghi"
}
```

`scripts/publish-atlas.sh` (nằm trong repo 9Link, không nằm ở đây) **chỉ ghi `SOURCE.json` sau khi
cổng tài liệu của trang thoát 0**. Vì vậy workflow ở repo này không cần cài lại cổng, cũng không
cần định nghĩa lại một phép kiểm nào: nó chỉ việc so `md5(index.html)` với `page_md5`. Nếu khớp
thì bytes đang được đăng **là** bytes đã qua cổng — suy ra bắc cầu, một định nghĩa, không bản sao
thứ hai của định nghĩa đó.

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

## Giấy phép — CHƯA quyết định, và repo này cố tình chưa có file `LICENSE`

White paper và tài liệu của 9Link nằm dưới **AGPL-3.0** (với carve-out cho `ee/`). Trang này là tác
phẩm phái sinh từ tài liệu đó. Repo này **chưa** có `LICENSE` — theo mặc định của GitHub nghĩa là
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

Script từ chối xuất bản nếu cổng đỏ, nếu thiếu `jsdom` (cổng thoát 3 = *bỏ qua*, và bỏ qua thì
không được coi là đạt), hoặc nếu `docs/atlas/index.html` còn thay đổi chưa commit — vì khi đó
`source_sha` sẽ không trỏ tới đúng bytes đang được đăng.
