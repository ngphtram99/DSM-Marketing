# Tra cứu Giá trị Cấp mẫu DSM

Trang web tĩnh (1 file `index.html`, không cần server/backend) đọc dữ liệu **trực tiếp** từ Google Sheet mỗi khi có người mở trang, cho phép tìm khách hàng và xem:

- Định mức, tồn đầu kỳ/cuối kỳ, đã xuất, vượt định mức (MEN & ĐÁ) — lấy từ tab `REPORT V2.`, phản ánh đúng tháng đang được chọn trên sheet đó.
- Lịch sử xuất hàng theo từng ngày (cột **O – Ngày tính DSM**), gộp theo tháng để so sánh — tự tổng hợp từ tab `Data_Update mỗi ngày_Theo Odoo`, không giới hạn 1 tháng.
- Tìm kiếm toàn bộ: theo tên khách hàng, mã đơn hàng, ghi chú, kho...

## Bước 1 — Cấp quyền xem cho Google Sheet

Vì trang không có backend/đăng nhập, sheet cần ở chế độ ai có link cũng xem được:

1. Mở Google Sheet → nút **Chia sẻ** (Share)
2. Chọn **"Bất kỳ ai có đường liên kết"** → quyền **Người xem (Viewer)**
3. Lưu lại

Sheet vẫn an toàn ở mức "chỉ ai có link mới xem" — không xuất hiện trên tìm kiếm Google, và không ai chỉnh sửa được.

## Bước 2 — Đưa code lên GitHub

```bash
git init
git add index.html README.md
git commit -m "DSM lookup tool"
git branch -M main
git remote add origin https://github.com/<tai-khoan-cua-ban>/dsm-tracker.git
git push -u origin main
```

(Hoặc tạo repo mới trên github.com rồi kéo-thả file `index.html` vào qua giao diện web, cũng được.)

## Bước 3 — Deploy bằng Vercel

1. Vào https://vercel.com → **Add New → Project**
2. Chọn **Import** repo GitHub vừa tạo
3. Vercel tự nhận đây là site tĩnh, không cần cấu hình gì thêm (không có Build Command, Output Directory để trống hoặc `.`)
4. Bấm **Deploy**

Xong, bạn sẽ có 1 link dạng `https://dsm-tracker.vercel.app` dùng được trên điện thoại/máy tính, mỗi lần mở sẽ tự đọc dữ liệu mới nhất từ sheet.

## Bước 4 (tuỳ chọn nhưng nên làm) — Apps Script để xem đúng MEN/ĐÁ theo mọi tháng

Mặc định, bảng **MEN/ĐÁ** (Tồn đầu kỳ, Định mức, Đã xuất...) chỉ đúng cho **đúng 1 tháng đang chọn sẵn trên dropdown "Tháng" của tab `REPORT V2.`** — vì đó là số do công thức trong sheet tính, web chỉ đọc lại chứ không tự tính được.

Làm theo các bước dưới đây để web **tự động** lấy đúng số của bất kỳ tháng nào bạn lọc, không cần tự tay đổi dropdown nữa:

1. Mở Google Sheet → **Tiện ích (Extensions) → Apps Script**.
2. Xoá nội dung mặc định trong file `Code.gs`, dán toàn bộ nội dung file **`AppsScript_Code.gs`** (đi kèm trong repo này) vào.
3. Bấm **Lưu** (biểu tượng đĩa mềm).
4. Bấm **Triển khai (Deploy) → New deployment**.
   - Chọn loại: **Web app**
   - Execute as: **Me (tài khoản của bạn)**
   - Who has access: **Anyone**
   - Bấm **Deploy**, cấp quyền (Authorize) khi được hỏi.
5. Sau khi deploy xong, copy **URL Web app** (dạng `https://script.google.com/macros/s/xxxxx/exec`).
6. Mở lại file `index.html`, tìm dòng:
   ```js
   const APPS_SCRIPT_URL = "";
   ```
   Dán URL vừa copy vào giữa 2 dấu ngoặc kép, ví dụ:
   ```js
   const APPS_SCRIPT_URL = "https://script.google.com/macros/s/xxxxx/exec";
   ```
7. Lưu file, đẩy lại lên GitHub — Vercel sẽ tự deploy bản mới.

**Cách hoạt động:** mỗi khi bạn chọn xem 1 tháng khác trên web, web sẽ gọi vào Apps Script này; script sẽ **tạm thời** đổi dropdown "Tháng"/"Miền" trên sheet, đợi công thức tính lại, đọc kết quả, rồi **trả dropdown về giá trị cũ ngay lập tức** để không ảnh hưởng người khác đang mở sheet trực tiếp. Kết quả được lưu tạm trên trình duyệt (cache) nên xem lại cùng tháng đó lần sau sẽ nhanh, không phải gọi lại.

**Lưu ý bảo mật:** vì Web app cần đặt quyền truy cập "Anyone" để website tĩnh gọi được (không cần đăng nhập Google), về lý thuyết bất kỳ ai có URL này đều có thể gọi vào và khiến sheet đổi dropdown trong chốc lát (dù luôn được trả về ngay sau đó). Script không cho phép chỉnh sửa gì khác ngoài 2 ô dropdown đó. Nếu muốn chặt chẽ hơn, bạn có thể đổi "Who has access" thành hạn chế hơn, nhưng khi đó cần thêm bước xác thực phức tạp hơn khi gọi từ web tĩnh.

Nếu không muốn làm bước này, cứ để `APPS_SCRIPT_URL = ""` — web vẫn chạy bình thường như cũ, chỉ là bảng MEN/ĐÁ sẽ hiện cảnh báo khi bạn lọc sang tháng khác tháng đang chọn trên sheet.

## Nếu cần chỉnh sửa sau này

Toàn bộ cấu hình nằm ở đầu phần `<script>` trong `index.html`:

```js
const SHEET_ID = "..."; // ID lấy từ link sheet
const RAW_SHEET_NAME    = "Data_Update mỗi ngày_Theo Odoo";
const REPORT_SHEET_NAME = "REPORT V2.";
```

Nếu bạn đổi tên tab, thêm/bớt cột trong 2 tab này, chỉ cần sửa lại các số thứ tự cột trong object `RAW` và `REP` ngay bên dưới cho khớp.

## Lưu ý quan trọng

- Bảng **MEN / ĐÁ** (định mức, tồn kho...) mặc định chỉ phản ánh đúng **tháng đang chọn ở ô dropdown trên sheet `REPORT V2.`** tại thời điểm tải trang. Làm theo **Bước 4** ở trên (Apps Script) để web tự lấy đúng số của mọi tháng bạn lọc.
- Phần **Lịch sử xuất hàng** thì không bị giới hạn — tự tính từ dữ liệu thô nên xem được mọi tháng, không phụ thuộc dropdown, không cần Apps Script.
