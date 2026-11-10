# Khác biệt giữa 9 repo — sổ cập nhật CHUNG

> ## ⚠️ Đọc ba điều này trước khi sửa file
>
> **1 · File này do NGƯỜI sửa. `publish-atlas.sh` tạo nó MỘT LẦN rồi không bao giờ đụng tới nữa.**
> Nó **không** nằm trong `cmp_manifest` của `SOURCE.json`, **không** bị kiểm md5, và bước *Kiểm toàn
> vẹn* của workflow **in ra một dòng nói thẳng là nó đang bỏ qua file này** — để không ai đọc log rồi
> tưởng đó là một chỗ bỏ sót. Danh sách những file thuộc lane người sửa nằm ở khoá `cmp_human_files`.
>
> **2 · File này KHÔNG có cổng ngôn ngữ nào gác.** Repo 9Link chạy `GATE-VNDOC` trên `docs/` của nó;
> file này nằm ở repo công khai và là bản GỐC (không có bản nào trong 9Link để mà gác). Hệ quả: một
> link tương đối chết, một khối Mermaid hỏng, hay một ký tự CJK lọt vào đây thì **không gì báo đỏ**.
> Đây là giá phải trả của việc cho phép sửa tay ở repo công khai, và nó được khai ra ở đây thay vì để
> người đọc tự phát hiện.
>
> **3 · Số liệu có thẩm quyền nằm ở `survey.json`, KHÔNG nằm trong file này.** Khối ảnh chụp ở §1 là
> bản do máy sinh **tại một thời điểm**, có khai SHA và ngày; nó **không được làm mới** khi lượt khảo
> sát mới chạy. Nếu §1 và `survey.json` cãi nhau, **`survey.json` thắng** — vì nó là file máy sinh và
> được md5-stamp, còn §1 là một ảnh chụp đã đông lạnh.
>
> **4 · `survey.json` nằm cạnh file này là BẢN ĐÃ LƯỢC, không phải bản nguyên văn trong repo nguồn.**
> Bản gốc chứa đường dẫn thư mục tạm trên máy của người chạy bộ đo, và tên thư mục đó là một hash theo
> tài khoản hệ điều hành — thứ không được phép vào lịch sử git công khai. `scripts/redact-survey.mjs`
> thay mỗi đường dẫn đó bằng một giá trị ổn định, **giữ lại tên repo** vì đó là phần có nghĩa. Số
> đường dẫn đã lược được khai trong `SOURCE.json` ở khoá `cmp_redacted_paths`. Việc lược được chứng
> minh **không đổi một con số nào**: lúc xuất bản, bảng §3.1 sinh từ bản đã lược phải giống hệt từng
> byte bảng sinh từ bản gốc, nếu không thì việc xuất bản dừng lại. Nên mọi con số bạn đọc ở đây và ở
> `survey.json` bên cạnh đều là số đo thật, không bị lược bớt.

**Sinh lần đầu:** `2026-11-10T18:35:16Z` · từ 9Link `b5c22d103cbdfe44a6b9ec072898abdae164ec71` · bằng `scripts/seed-compare.mjs`.

---

## 1. Ảnh chụp bảng đo 18 chiều — MÁY SINH, ĐÔNG LẶNG

Đo lúc **`2026-11-10T10:38:36Z`**. Cửa sổ độ sống: `**survey.json lượt này KHÔNG có khoá `window_since_utc`** — nó được thêm vào `scripts/survey-9repo.sh` SAU khi lượt này chạy, nên con số đó chỉ có ở lượt kế tiếp. Không tự suy ra "bây giờ trừ 90 ngày": một cửa sổ tính lại mỗi lần chạy thì không kiểm chứng được, và một lời cảnh báo phụ thuộc vào NGÀY NGƯỜI TA ĐỌC nó thì vô nghĩa.`.

Bảng dưới đây là **output nguyên trạng** của `node scripts/survey-to-md.mjs survey.json` tại thời điểm
file này được tạo lần đầu — không một chữ nào được gõ tay, và không một chữ nào được làm mới về sau.
Output đó giống hệt từng byte dù sinh từ bản gốc hay từ bản đã lược, và sự giống hệt đó được kiểm lúc
xuất bản chứ không phải một lời hứa.
Nó nằm giữa hai marker `SEED-TABLE` để ai cũng thấy ranh giới của phần máy sinh.

⚠️ Bảng đó tham chiếu **§2.2** và **§3.2** — đó là các mục của `docs/so-sanh-cac-repo.md`, nằm trong
repo **private** `khanhlegood3/9Link` nên **không mở được từ đây**. Định nghĩa của từng cột vì thế không
đọc được ở repo công khai; đó là một giới hạn có thật, được khai ra chứ không giấu. Muốn đọc định
nghĩa thì phải mở repo nguồn.

<!-- SEED-TABLE BEGIN — máy sinh, đông lạnh tại SHA ở đầu file. KHÔNG làm mới, KHÔNG sửa tay. -->
<!-- §3.1 BEGIN — máy sinh: node scripts/survey-to-md.mjs · KHÔNG sửa tay phần ở giữa hai marker -->

Đo lúc **2026-11-10T10:38:36Z** bằng `scripts/survey-9repo.sh` (bộ đo của repo: `repo-measure.mjs` + `loc.py`).
**9** hàng · 9 đo được, 0 chưa · mỗi hàng đo được mang SHA của chính nó, vì chín hàng là chín cây khác nhau đo ở chín thời điểm khác nhau trong cùng một lượt chạy.

| Repo | Quyền đo được | Private | Độ sống (commit/90 ngày) | Ngôn ngữ chính | Quy mô (LOC src) | Data layer | Deploy | Quan hệ với 9Link |
|---|---|---|---|---|---|---|---|---|
| **`9Link`** @ `da6f7d1` | ✅ đọc + push | có | **470** | TypeScript · đuôi nhiều dòng nhất: `.ts` | **87.749** src / 555 file · 50.703 unit · tỉ lệ 0.58 · 36.228 e2e <br>*(riêng `.ts/.tsx`: src 72.457 — hai phạm vi, hai số khác nhau)* | Prisma **34** model · 34 `@@map` · 55 migration · postgresql · postgres + clickhouse + redis + mongo | **4** compose · lớn nhất `docker-compose.yml` 9 svc (6 `profiles`) <br>[postgres · clickhouse · clickhouse-uae · clickhouse-eu · postgres-uae · postgres-eu · redis · growthbook · growthbook-mongo] <br>còn lại: `docker-compose.yml` 6 svc · `docker-compose.app.yml` 2 svc · `docker-compose.demo.yml` 6 svc <br>start ["node", "apps/web/server.js"] | — *(chính nó; chi tiết ở §3.2)* |
| **`9Link-V2`** @ `c97f082` | ✅ đọc + push | có | **68** | TypeScript · đuôi nhiều dòng nhất: `.ts` | **21.059** src / 103 file · 8.553 unit · tỉ lệ 0.41 · 0 e2e <br>*(riêng `.ts/.tsx`: src 19.487 — hai phạm vi, hai số khác nhau)* | Prisma **8** model · 8 `@@map` · 2 migration · postgresql · postgres + clickhouse | **1** compose · lớn nhất `docker-compose.yml` 3 svc <br>[postgres · clickhouse · redis] | **chưa kết luận.** Bằng chứng: `package.json` name = **`9link-v2`**; **44** đường dẫn trùng với 9Link (repo này 206 file / 9Link 1084 file); mẫu: `.claude/agents/architect.md`, `.claude/agents/doc-writer.md`, `.claude/agents/implementer.md`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |
| **`9Link-V3`** @ `35a3fef` | ✅ đọc + push | có | **2** | PowerShell · đuôi nhiều dòng nhất: `.py` | **450** src / 4 file · 207 unit · tỉ lệ 0.46 · 0 e2e <br>*(riêng `.ts/.tsx`: src 31 — hai phạm vi, hai số khác nhau)* | không có ORM được phát hiện | không có file compose | **chưa kết luận.** Bằng chứng: không có `package.json` ở gốc; **11** đường dẫn trùng với 9Link (repo này 95 file / 9Link 1084 file); mẫu: `.claude/agents/architect.md`, `.claude/agents/implementer.md`, `.gitignore`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |
| **`9Link-new`** @ `0995bd2` | ✅ đọc + push | có | **6** | TypeScript · đuôi nhiều dòng nhất: `.ts` | **33.418** src / 222 file · 8.786 unit · tỉ lệ 0.26 · 0 e2e <br>*(riêng `.ts/.tsx`: src 29.712 — hai phạm vi, hai số khác nhau)* | không có ORM được phát hiện | **1** compose · lớn nhất `compose.yaml` 2 svc <br>[postgres · redis] | **chưa kết luận.** Bằng chứng: `package.json` name = **`9link`**; **10** đường dẫn trùng với 9Link (repo này 461 file / 9Link 1084 file); mẫu: `.dockerignore`, `.env.example`, `.gitignore`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |
| **`9link-browser`** @ `976bea5` | ✅ đọc + push | có | **12** | PowerShell *(API)* | **0** src / 0 file · 0 unit · tỉ lệ n/a (src = 0) · 0 e2e <br>*(riêng `.ts/.tsx`: src 0 — hai phạm vi, hai số khác nhau)* | không có ORM được phát hiện | không có file compose | **chưa kết luận.** Bằng chứng: không có `package.json` ở gốc; **4** đường dẫn trùng với 9Link (repo này 11 file / 9Link 1084 file); mẫu: `.gitignore`, `CLAUDE.md`, `DECISIONS.md`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |
| **`9do`** @ `4dbc2ff` | ✅ đọc + push | có | **3** · *pushed_at sớm hơn created_at ⇒ mốc thời gian kế thừa từ repo cha* | đuôi `.ts` *(không có `language` từ API)* | **53.751** src / 430 file · 2.659 unit · tỉ lệ 0.05 · 0 e2e <br>*(riêng `.ts/.tsx`: src 50.667 — hai phạm vi, hai số khác nhau)* | Prisma **73** model · 73 `@@map` · 19 migration · postgresql · postgres + redis + minio | **5** compose · lớn nhất `docker-compose.yml` 7 svc <br>[postgres · redis · migrate · api · web · business · admin] <br>còn lại: `docker-compose.demo.yml` 7 svc · `docker-compose.prod.yml` 5 svc · `docker-compose.test.yml` 1 svc · `docker-compose.yml` 4 svc <br>mạng ngoài: edge <br>start ["npm", "run", "start"] | **chưa kết luận.** Bằng chứng: `package.json` name = **`9do-v2`**; **20** đường dẫn trùng với 9Link (repo này 596 file / 9Link 1084 file); mẫu: `.dockerignore`, `.env.example`, `.gitignore`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |
| **`olacity`** @ `b18d49d` | ✅ đọc + push | có | **38** | TypeScript · đuôi nhiều dòng nhất: `.ts` | **52.361** src / 420 file · 2.316 unit · tỉ lệ 0.04 · 0 e2e <br>*(riêng `.ts/.tsx`: src 49.488 — hai phạm vi, hai số khác nhau)* | Prisma **69** model · 69 `@@map` · 16 migration · postgresql · postgres + redis + minio | **5** compose · lớn nhất `docker-compose.yml` 7 svc <br>[postgres · redis · migrate · api · web · business · admin] <br>còn lại: `docker-compose.demo.yml` 7 svc · `docker-compose.prod.yml` 5 svc · `docker-compose.test.yml` 1 svc · `docker-compose.yml` 4 svc <br>mạng ngoài: edge <br>start ["npm", "run", "start"] | **chưa kết luận.** Bằng chứng: `package.json` name = **`olacity-v2`**; **20** đường dẫn trùng với 9Link (repo này 582 file / 9Link 1084 file); mẫu: `.dockerignore`, `.env.example`, `.gitignore`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |
| **`olacity-v1`** @ `f0ebec7` | ✅ đọc + push | có | **0** ⚠ *0 mà `pushed_at` = 2026-09-07 nằm trong cửa sổ — thường vì fork kế thừa `pushed_at` của repo cha, hoặc lời gọi API hỏng (script ghi `int(n90 or 0)` nên "đo được 0" và "không đo được" CÙNG ra 0)* · *pushed_at sớm hơn created_at ⇒ mốc thời gian kế thừa từ repo cha* | đuôi `.sql` *(không có `language` từ API)* | **42** src / 1 file · 0 unit · tỉ lệ 0.00 · 0 e2e <br>*(riêng `.ts/.tsx`: src 0 — hai phạm vi, hai số khác nhau)* | không có ORM được phát hiện | **1** compose · lớn nhất `docker-compose.yml` 1 svc <br>[mysql] <br>mạng ngoài: olacity-mysql-data | **chưa kết luận.** Bằng chứng: không có `package.json` ở gốc; **2** đường dẫn trùng với 9Link (repo này 10 file / 9Link 1084 file); mẫu: `.gitignore`, `HANDOFF.md`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |
| **`olacity-docs-web`** @ `f9448e4` | ✅ đọc + push | có | **0** · *pushed_at sớm hơn created_at ⇒ mốc thời gian kế thừa từ repo cha* | đuôi `.tsx` *(không có `language` từ API)* | **2.310** src / 30 file · 0 unit · tỉ lệ 0.00 · 0 e2e <br>*(riêng `.ts/.tsx`: src 2.066 — hai phạm vi, hai số khác nhau)* | không có ORM được phát hiện | **1** compose · lớn nhất `docker-compose.yml` 1 svc <br>[web] <br>mạng ngoài: edge <br>start ["node", "server.js"] | **chưa kết luận.** Bằng chứng: `package.json` name = **`olacity-docs-web`**; **5** đường dẫn trùng với 9Link (repo này 41 file / 9Link 1084 file); mẫu: `.dockerignore`, `.gitignore`, `deploy/deploy.json`. *Chưa diff NỘI DUNG các file trùng tên nên chưa phân biệt được fork / viết lại / thử nghiệm bỏ / nhánh song song* |

> Định nghĩa của từng cột ở **§2.2** — cố tình KHÔNG chép lại ở đây, vì một định nghĩa tồn tại ở hai
> chỗ là một định nghĩa sớm muộn cũng cãi nhau. Hai điểm cần nhớ khi đọc bảng: **(a)** cột "Độ sống"
> là số commit 90 ngày, *không phải số sao*; **(b)** cột "Quy mô" là MỌI đuôi nguồn của §2.3, và số
> `.ts/.tsx` được in riêng ngay trong ô vì hai phạm vi cho hai số khác nhau.
> Cột **"Quan hệ với 9Link" cố ý để trống kể cả khi tên repo gợi ý.** `9Link-V2` / `V3` / `-new` nghe như
> ba thế hệ của cùng một sản phẩm, nhưng **ba repo cùng tên họ có thể là**: fork · viết lại · bản thử
> nghiệm bị bỏ · hoặc ba nhánh tính năng song song. Kết luận sai ở cột này sẽ làm hỏng cả bảng, nên nó
> **chỉ** được điền sau khi so `package.json`, cây thư mục và vài file trùng tên. Cột đó **chỉ in bằng chứng**
> và sẽ tự đổi thành kết luận khi `survey-9repo.sh` đo thêm được `relation.content_diff` — tức diff NỘI
> DUNG vài file trùng tên, việc mà một lượt so đường dẫn chưa làm được.

<!-- §3.1 END -->
<!-- SEED-TABLE END -->

---

## 2. Phần NGƯỜI cập nhật — khác biệt mà máy KHÔNG đo được

Máy chỉ đo được **18 chiều cấu trúc** (file, LOC, ORM, compose, CI, license…). Nó **không** đọc được mã
nguồn của 8 repo anh em: cả 8 trả `403 Resource not accessible by integration` tại thời điểm đo, có đối
chứng `404` cho một repo không tồn tại. Nên **mọi kết luận về TÍNH NĂNG** — repo nào có cái gì, cái
nào tốt hơn, cái nào nên học — là phần của con người, và nó thuộc về §2 này.

**Quy tắc khi điền, lấy từ chính luật của repo nguồn:**

1. **Mỗi con số phải mang SHA + ngày + định nghĩa.** Một con số không có ba thứ đó thì không kiểm lại
   được, và sẽ không ai biết nó còn đúng không.
2. **Không đoán.** Chưa đo được thì viết rõ *"(chưa đo được)"* kèm lý do — một ô trống im lặng sẽ được
   người đọc tự suy ra, và họ sẽ suy ra sai.
3. **Một phép đo chỉ sống ở một nơi.** Nếu một con số đã có trong `survey.json` thì **trỏ tới đó**,
   đừng chép lại. §1 là ngoại lệ duy nhất, và nó được phép vì đã tự khai là ảnh chụp đông lạnh.
4. **AGPL-3.0 là ràng buộc LAN.** "Repo kia làm hay hơn, chép sang" là một **quyết định pháp lý**, không
   phải một thao tác `cp`. Ghi rõ ai đã quyết, ngày nào.

### 2.1 `9Link-V2` @ `c97f082`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `9Link-V2`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `9Link-V2` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |

### 2.2 `9Link-V3` @ `35a3fef`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `9Link-V3`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `9Link-V3` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |

### 2.3 `9Link-new` @ `0995bd2`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `9Link-new`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `9Link-new` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |

### 2.4 `9link-browser` @ `976bea5`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `9link-browser`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `9link-browser` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |

### 2.5 `9do` @ `4dbc2ff`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `9do`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `9do` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |

### 2.6 `olacity` @ `b18d49d`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `olacity`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `olacity` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |

### 2.7 `olacity-v1` @ `f0ebec7`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `olacity-v1`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `olacity-v1` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |

### 2.8 `olacity-docs-web` @ `f9448e4`

Số MÁY đo được của repo này: mở `survey.json` cùng thư mục, hàng có `repo` = `olacity-docs-web`.
**Đừng chép số đó vào đây** — chép là tạo một bản sao thứ hai của cùng một phép đo trong cùng
một thư mục, và lượt khảo sát kế tiếp sẽ làm hai bản cãi nhau mà không cổng nào bắt được.

⚠️ `relation.content_diff` đang `null` cho **cả 9/9** hàng, tức CHƯA AI diff nội dung
các file trùng tên. Vậy việc ĐẦU TIÊN của lane này là diff nội dung vài file trùng tên, **không phải**
kết luận "fork / viết lại / nhánh song song" — hai repo cùng tên họ có thể là bất kỳ cái nào trong ba.

| Chiều khác biệt | 9Link | `olacity-docs-web` | Bằng chứng (SHA + ngày + định nghĩa) | Ai quyết, ngày nào |
|---|---|---|---|---|
| _(chưa điền)_ | | | | |


---

## 3. Nhật ký cập nhật

Mỗi lần sửa §2, thêm MỘT dòng ở đây: ngày (UTC) · người sửa · repo vừa vào đọc · tóm tắt một dòng.
Không có nhật ký thì không ai phân biệt được "chưa ai đo" với "đã đo và thấy không có gì".

| Ngày (UTC) | Người | Repo vừa đọc | Đã cập nhật gì |
|---|---|---|---|
| _(chưa có lượt nào)_ | | | |
