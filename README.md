
# Hướng dẫn cài đặt và sử dụng Telegram Bot quản lý tài chính
![Banner](banner.png)
---
## UPDATES
 ### Tách dòng gửi tin nhắn khi báo cáo quá nhiều giao dịch 
  From:
 (https://github.com/htcminh)
 ### Hạn chế gửi tin nhắn báo sai cú pháp trong nhóm nhiều người.
  Bỏ qua các tin nhắn không có mục đích tương tác với bot.
  
  Khi nhập đúng cú pháp thu/chi hoặc các lệnh bot mới phản hồi, nhập cú pháp giao dịch, các lệnh nhưng sai cú pháp bot vẫn sẽ phản hồi.
 
 ### Quản lý danh sách người dùng được phép sử dụng bot

 Phục vui cho việc quản lý hội nhóm được thuận tiện!
 
 Chỉ admin mới có quyền thêm/xóa người dùng bằng /addusers và /delusers.
 
 UserID là id Telegram của bạn.
 
 UserID sẽ được lưu vào sheet riêng.
 
 **Lệnh mới**

    "/addusers <user_id>" (Thêm người dùng vào danh sách được phép)

    "/delusers <user_id>" (Xóa người dùng khỏi danh sách)

---
## 1. Giới thiệu
Telegram bot giúp bạn quản lý tài chính cá nhân, lưu trữ dữ liệu trên Google Sheets và cung cấp báo cáo theo thời gian. 

Bạn có thể:
- Thêm giao dịch thu/chi.
- Xem báo cáo theo tuần, tháng.
- Xóa giao dịch gần nhất hoặc toàn bộ dữ liệu.

---

## 2. Cài đặt

### 2.1. Tạo Telegram Bot
1. Mở ứng dụng Telegram, tìm kiếm **BotFather**.
2. Gửi lệnh `/newbot` và làm theo hướng dẫn để tạo bot mới.
3. Sau khi hoàn tất, bạn sẽ nhận được **TOKEN** để kết nối bot.

### 2.2. Tạo Google Sheets
1. Truy cập Google Sheets và tạo một bảng tính mới.
2. Đổi tên sheet (ví dụ: Finance Data).
3. Tạo các cột(Không bắt buộc): **Thời gian**, **Loại**, **Số tiền**, **Mô tả**.
4.Lấy Sheet ID từ URL

  
	Ví dụ URL:

https://docs.google.com/spreadsheets/d/1A2B3C4D5E6F7G8H9I0J/edit#gid=0

 Sheet ID là phần:     **1A2B3C4D5E6F7G8H9I0J**

5. Lấy ADMIN ID (Để sử dụng tính năng add người có quyền dùng bot)
   
   Chính là dãy số id tài khoản Telegram của bạn, nếu có nhiều hơn 1 admin thì cách nhau bằng dấu phẩy và nằm trong ngoặc kép "
   
### 2.3. Triển khai Google Apps Script
1. Mở Google Sheets > Extensions > Apps Script.
2. Lấy mã nguồn:
   > [!IMPORTANT]
   > Hãy mở file [**Code**] trong thư mục dự án này, copy toàn bộ nội dung và dán vào cửa sổ biên tập Apps Script (xóa hết code cũ nếu có).

 ### 2.4 Thay thế:
   - `YOUR_TELEGRAM_BOT_TOKEN` bằng token bot Telegram.
   - `YOUR_SHEET_ID` bằng ID Google Sheets.
   - `ADMIN_IDS` là các id tài khoản telegram mà bạn muốn làm admin.

 ### 2.5 Triển khai (Sau khi dán mã code và thay thế các giá trị)
 
 **Deploy** → **New deployment** → **Web app**

 Hoặc
 
  **Triển khai** -> **Tuỳ chọn triển khai mới** 
   - **Chọn loại**: Ứng dụng web
   - **Thực thi bằng tên**: Tôi  
   - **Người có quyền truy cập**: Bất kỳ ai
   -  Sau đó nhấn triển khai và cấp quyền  
 #### Lấy **Web App URL** sau khi triển khai(copy cả đoạn link nhé).

### 2.6 Cấu Hình Webhook**
1. Quay lại file code trong Apps Script.
2. Dán URL vừa copy vào biến `WEB_APP_URL` ở đầu file.
3. Lưu lại (Ctrl + S).
4. Ở thanh công cụ phía trên, chọn hàm `setWebhook` và nhấn **Run (Chạy)**.
5. Kiểm tra nhật ký thực thi, nếu hiện "Success" là bot đã sẵn sàng!
## 3. Sử dụng

### 3.1. Bắt đầu sử dụng bot
Gửi lệnh `/start` để nhận hướng dẫn cơ bản.

### 3.2. Thêm giao dịch
Nhập giao dịch theo cú pháp:
```
<số tiền> <thu/chi> <mô tả>
```

### 3.3. Xem báo cáo
- **Báo cáo tổng:** `/report`
- **Báo cáo tháng:** `/report 01/2025`
- **Báo cáo tuần:** `/report 04/01/2025`
- **Sắp xếp tăng/giảm:** Thêm `az` (tăng) hoặc `za` (giảm).
  - Ví dụ: `/report az`, `/report 01/2025 za`.

### 3.4 Admin có thể thêm/xóa người dùng bằng lệnh:
- **Thêm user vào danh sách:**/addusers <user_id> 
- **Xóa user khỏi danh sách:**/delusers <user_id>
- **Ví dụ:**/addusers 999999999
- Danh sách user được lưu trong sheet users và không bị mất khi reset dữ liệu.
  
#### Ví dụ chi tiết:
1. Xem toàn bộ giao dịch, sắp xếp tăng dần: `/report az`
2. Báo cáo chi tiêu tháng 1 năm 2025: `/report 01/2025`
3. Báo cáo tuần chứa ngày 04/01/2025: `/report 04/01/2025 za`.

### 3.4. Xóa giao dịch
- **Xóa giao dịch gần nhất:** Gửi lệnh `/undo`.
- **Xóa toàn bộ dữ liệu:** Gửi lệnh `/reset`.

---

## 4. Ví dụ cụ thể

### Thêm giao dịch
- Thu nhập: `13058k thu Tiền thưởng cuối năm`
- Chi tiêu: `69k chi mua dầu ăn`

### Báo cáo chi tiết
1. Báo cáo tổng, sắp xếp theo thứ tự giảm dần:
   ```
   /report za
   ```
   Kết quả:
   ```
   Báo cáo tổng:
   Tổng thu: 1,000,000 VND
   Tổng chi: 300,000 VND
   Cân đối: 700,000 VND

   Giao dịch thu nhập cụ thể:
   + 1,000,000 VND: Tiền thưởng cuối năm (01/01/2025 10:00)

   Giao dịch chi tiêu cụ thể:
   - 300,000 VND: Mua thực phẩm (01/01/2025 14:00)
   ```

2. Báo cáo tháng 1/2025:
   ```
   /report 01/2025
   ```

---

## 5. Lưu ý

*Quy ước: 1k = 1000VND, 1tr = 1000000VND*

*Không nhập 5tr2 hoặc lẻ, nếu lẻ thì nhập 5215k*

*Google Sheets không được xóa hoặc thay đổi ID.*
 
*Tài khoản Gmail cần cấp quyền cho Google Sheets khi cài Webhook.*
 
*Đảm bảo bot Telegram đã được kết nối đúng Webhook.*
- **Lỗi Webhook:** Đảm bảo bạn đã chạy hàm `setWebhook` sau khi dán URL Web App.
- **Quyền truy cập:** Khi triển khai, bắt buộc phải chọn "Anyone" để Telegram có thể gửi dữ liệu về.
- **Định dạng tiền:** Nhập số nguyên, không dùng dấu chấm/phẩy giữa các con số (ví dụ dùng `1500k` thay vì `1.500.000`).
- **Không lưu dữ liệu:** Kiểm tra Sheet ID và quyền truy cập.

## Liên hệ

| Kênh | Link |
|------|------|
| GitHub | [nguyenngocphung2000](https://github.com/nguyenngocphung2000) |
| Telegram | [@nothing3272](https://t.me/nothing3272) |
| Facebook | [Nguyễn Ngọc Phụng](https://www.facebook.com/share/1Ayyxg5kjH/) |
| Email Form | [Google Form](https://forms.gle/5brLdS34QMQ3ei157) |
---
