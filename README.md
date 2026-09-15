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

## Nếu cần chỉnh sửa sau này

Toàn bộ cấu hình nằm ở đầu phần `<script>` trong `index.html`:

```js
const SHEET_ID = "..."; // ID lấy từ link sheet
const RAW_SHEET_NAME    = "Data_Update mỗi ngày_Theo Odoo";
const REPORT_SHEET_NAME = "REPORT V2.";
```

Nếu bạn đổi tên tab, thêm/bớt cột trong 2 tab này, chỉ cần sửa lại các số thứ tự cột trong object `RAW` và `REP` ngay bên dưới cho khớp.

## Lưu ý quan trọng

- Bảng **MEN / ĐÁ** (định mức, tồn kho...) chỉ phản ánh đúng **tháng đang chọn ở ô dropdown trên sheet `REPORT V2.`** tại thời điểm tải trang — vì đó là số đã được các công thức trong sheet tính sẵn. Muốn xem tháng khác, cần đổi dropdown "Tháng" trên chính Google Sheet trước.
- Phần **Lịch sử xuất hàng** thì không bị giới hạn — tự tính từ dữ liệu thô nên xem được mọi tháng, không phụ thuộc dropdown.
