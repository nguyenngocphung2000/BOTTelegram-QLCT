# Hướng dẫn cài đặt và sử dụng Telegram Bot quản lý tài chính

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
1. Tạo một bảng tính mới trên Google Sheets.
2. Thêm tiêu đề các cột: `Thời gian`, `Loại`, `Số tiền`, `Mô tả`.
3. Lấy **ID của bảng tính** từ URL:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit
   ```

### 2.3. Triển khai Google Apps Script
1. Mở Google Sheets > Extensions > Apps Script.
2. Dán mã sau:
   [Code](https://raw.githubusercontent.com/nguyenngocphung2000/BOTTelegram-QLCT/refs/heads/main/Code)

3. Thay thế:
   - `YOUR_TELEGRAM_BOT_TOKEN` bằng token bot Telegram.
   - `YOUR_SHEET_ID` bằng ID Google Sheets.
4. Lưu mã và triển khai:
   - Chọn **Deploy** > **New Deployment**.
   - Chọn loại **Web App**.
   - Đặt quyền **Anyone** can execute.
   - Sao chép URL triển khai.

### 2.4. Kết nối Webhook
1. Kết nối bot với webhook:
   ```
   https://api.telegram.org/bot<TOKEN>/setWebhook?url=<DEPLOYMENT_URL>
   ```
   Thay `<TOKEN>` và `<DEPLOYMENT_URL>` lúc bạn sao chép url triển khai.

---

## 3. Sử dụng

### 3.1. Bắt đầu sử dụng bot
Gửi lệnh `/start` để nhận hướng dẫn cơ bản.

### 3.2. Thêm giao dịch
Nhập giao dịch theo cú pháp:
```
<số tiền> <thu/chi> <mô tả>
```
#### Ví dụ:
- **Thu nhập:** `10tr thu Lương tháng 1`
- **Chi tiêu:** `256k chi Mua sách giáo khoa`

### 3.3. Xem báo cáo
- **Báo cáo tổng:** `/report`
- **Báo cáo tháng:** `/report 01/2025`
- **Báo cáo tuần:** `/report 04/01/2025`
- **Sắp xếp tăng/giảm:** Thêm `az` (tăng) hoặc `za` (giảm).
  - Ví dụ: `/report az`, `/report 01/2025 za`.

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
- Chi tiêu: `69 chi mua dầu ăn`

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

### Xóa giao dịch
- Xóa giao dịch gần nhất: `/undo`.
- Xóa tất cả dữ liệu: `/reset`.

---

## 5. Lưu ý

*Quy ước: 1k = 1000VND, 1tr = 1000000VND*

*Không nhập 5tr2 hoặc lẻ, nếu lẻ thì nhập 5200k*

*Google Sheets không được xóa hoặc thay đổi ID.*
 
*Tài khoản Gmail cần cấp quyền cho Google Sheets khi cài Webhook.*
 
*Đảm bảo bot Telegram đã được kết nối đúng Webhook.*

***Cần tư vấn liên hệ t.me/nothing3272***

---
