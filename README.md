# Hướng dẫn cài đặt và sử dụng Telegram Bot quản lý tài chính
---
## UPDATES
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
   Chính là id tài khoản Telegram của bạn, nếu có nhiều hơn 1 admin thì cách nhau bằng dấu phẩy và nằm trong ngoặc kép "
### 2.3. Triển khai Google Apps Script
1. Mở Google Sheets > Extensions > Apps Script.
2. Dán mã sau (nhớ xoá mã cũ đi):

```

const TOKEN = "YOUR_TELEGRAM_BOT_TOKEN";
const API_URL = `https://api.telegram.org/bot${TOKEN}`;
const SHEET_ID = "YOUR_SHEET_ID";
const ADMIN_IDS = ["123456789", "987654321"]; // Thay các dãy số ở trong bằng id telegram mà bạn muốn làm admin

function doPost(e) {
  const { message } = JSON.parse(e.postData.contents);
  const chatId = message.chat.id;
  const text = message.text;
  const userId = message.from.id;

  if (!isAuthorizedUser(userId)) {
    sendMessage(chatId, "🚫 Bạn không có quyền sử dụng bot này.");
    return;
  }

  if (text.startsWith("/start")) {
    sendStartMessage(chatId);
  } else if (text.startsWith("/addusers") || text.startsWith("/delusers")) {
    if (!isAdmin(userId)) {
      sendMessage(chatId, "🚫 Bạn không phải là admin.");
      return;
    }
    manageUsers(chatId, userId, text);
  } else {
    if (text.startsWith("/report")) {
      handleReport(chatId, text);
    } else if (text.startsWith("/reset")) {
      resetSheet(chatId);
    } else if (text.startsWith("/undo")) {
      undoLast(chatId);
    } else {
      handleTransaction(chatId, text);
    }
  }
}

function isAdmin(userId) {
  return ADMIN_IDS.includes(String(userId));
}

function isAuthorizedUser(userId) {
  const sheet = getOrCreateUserSheet();
  const lastRow = sheet.getLastRow();
  
  if (lastRow < 2) return ADMIN_IDS.includes(String(userId));
  const userIds = sheet.getRange(2, 1, lastRow - 1, 1).getValues().flat().map(String);
  return ADMIN_IDS.includes(String(userId)) || userIds.includes(String(userId));
}

function sendStartMessage(chatId) {
  ensureSheetsExist();

  const startMessage = `
*Chào mừng bạn đến với ứng dụng quản lý tài chính cá nhân!*\n\n` +
      `📌 *Hướng dẫn sử dụng:*\n\n` +
      `1️⃣ *Thêm giao dịch:*\n   _Nhập theo cú pháp:_ <số tiền> <thu/chi> <mô tả>.\n` +
        `   *Ví dụ:* \`14629k thu Lương t1\`\n\n` +
              `2. *Xem báo cáo:*\n` +
        `   - \`/report\`: Báo cáo tổng.\n` +
        `   - \`/report mm/yyyy\`: Báo cáo tháng.\n` +
        `   - \`/report dd/mm/yyyy\`: Báo cáo tuần (hiển thị tuần có ngày được chọn).\n` +
        `   - Thêm "az" hoặc "za" sau lệnh để sắp xếp:\n` +
        `     *Ví dụ:* \`/report az\` hoặc \`/report 01/2024 za\`\n\n` +
      `3️⃣ *Quản lý người dùng(chỉ Admin):*\n` +
      `   - \`/addusers <id>\`: _Thêm user._\n` +
      `   - \`/delusers <id>\`: _Xóa user._\n\n` +
      `4️⃣ *Khác:*\n` +
      `   - \`/undo\`: _Xóa giao dịch gần nhất._\n` +
      `   - \`/reset\`: _Xóa dữ liệu (trừ user)._\n\n` +
        `💡 *Lưu ý:*\n` +
        `- Số tiền có thể nhập dạng "1234k" (1,234,000) hoặc "1tr" (1,000,000).\n` +
      ;

  sendMessage(chatId, startMessage);
}

  function getOrCreateUserSheet() {
  const ss = SpreadsheetApp.openById(SHEET_ID);
  let usersSheet = ss.getSheetByName("users");

  if (!usersSheet) {
    usersSheet = ss.insertSheet("users");
    usersSheet.appendRow(["UserID"]);
  }

  return usersSheet;
}
function ensureSheetsExist() {
  const ss = SpreadsheetApp.openById(SHEET_ID);

  let transactionsSheet = ss.getSheetByName("transactions");
  if (!transactionsSheet) {
    transactionsSheet = ss.insertSheet("transactions");
    transactionsSheet.appendRow(["Thời gian", "Loại", "Số tiền", "Mô tả"]);
  }

  let usersSheet = ss.getSheetByName("users");
  if (!usersSheet) {
    usersSheet = ss.insertSheet("users");
    usersSheet.appendRow(["UserID"]);
  }
}
function handleTransaction(chatId, text) {
  const [amount, type, ...desc] = text.split(" ");
  if (!isValidAmount(amount) || !["thu", "chi"].includes(type.toLowerCase())) {
    sendMessage(chatId, "Lỗi: Nhập đúng cú pháp <số tiền> <thu/chi> <mô tả>.");
    return;
  }

  const description = desc.join(" ");
  const formattedDesc = description.charAt(0).toUpperCase() + description.slice(1);
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName("transactions");
  sheet.appendRow([new Date(), type.toLowerCase(), parseAmount(amount), formattedDesc || "Không có mô tả"]);
  sendMessage(chatId, `Đã thêm giao dịch:\nSố tiền: ${amount}\nLoại: ${type}\nMô tả: ${formattedDesc}`);
}

function manageUsers(chatId, userId, text) {
  const args = text.split(" ");
  const command = args[0];
  const targetUserId = args[1];

  if (!targetUserId) {
    sendMessage(chatId, "🚫 Bạn cần cung cấp ID người dùng.");
    return;
  }

  if (command === "/addusers") {
    addUser(chatId, targetUserId);
  } else if (command === "/delusers") {
    removeUser(chatId, targetUserId);
  } else {
    sendMessage(chatId, "🚫 Lệnh không hợp lệ.");
  }
}

function addUser(chatId, targetUserId) {
  const sheet = getOrCreateUserSheet();
  const lastRow = sheet.getLastRow();

  const existingUsers = lastRow > 1
    ? sheet.getRange(2, 1, lastRow - 1, 1).getValues().flat().map(String)
    : [];

  if (existingUsers.includes(targetUserId)) {
    sendMessage(chatId, `🚫 Người dùng ID ${targetUserId} đã có trong danh sách.`);
    return;
  }

  sheet.appendRow([targetUserId]);
  sendMessage(chatId, `✅ Đã thêm người dùng với ID ${targetUserId}.`);
}
function removeUser(chatId, targetUserId) {
  const sheet = getOrCreateUserSheet();
  const lastRow = sheet.getLastRow();

  if (lastRow < 2) {
    sendMessage(chatId, `🚫 Không có người dùng nào trong danh sách.`);
    return;
  }

  const userIds = sheet.getRange(2, 1, lastRow - 1, 1).getValues().flat().map(String);
  const userIndex = userIds.indexOf(String(targetUserId));

  if (userIndex === -1) {
    sendMessage(chatId, `🚫 Không tìm thấy người dùng với ID ${targetUserId}.`);
    return;
  }

  sheet.deleteRow(userIndex + 2);
  sendMessage(chatId, `✅ Đã xóa người dùng với ID ${targetUserId}.`);
}
function handleReport(chatId, text) {
  const dateRegex = /\d{2}\/\d{4}|\d{2}\/\d{2}\/\d{4}/;
  const dateParam = text.match(dateRegex)?.[0];
  let filter = "all";
  let sortOrder = null;

  if (text.includes("az")) {
    sortOrder = "az";
  } else if (text.includes("za")) {
    sortOrder = "za";
  }

  if (dateParam) {
    filter = dateParam.length === 7 ? "month" : "week";
  }

  generateReport(chatId, filter, dateParam, sortOrder);
}

function generateReport(chatId, filter, dateParam, sortOrder) {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getActiveSheet();
  const data = sheet.getDataRange().getValues().slice(1);

  if (!data.length) {
    sendMessage(chatId, "Không có dữ liệu.");
    return;
  }

  const now = parseDate(filter, dateParam);
  const filteredData = data.filter(([date]) =>
    isValidDate(new Date(date), filter, now)
  );

  if (sortOrder) {
    filteredData.sort((a, b) => {
      const amountA = a[2];
      const amountB = b[2];
      return sortOrder === "az" ? amountA - amountB : amountB - amountA;
    });
  }

  const incomeTransactions = [];
  const expenseTransactions = [];
  let [income, expense] = [0, 0];

  filteredData.forEach(([date, type, amount, desc]) => {
    const formattedReportDate = new Date(date).toLocaleString("vi-VN", {
      hour: "2-digit",
      minute: "2-digit",
      day: "2-digit",
      month: "2-digit",
      year: "numeric",
      hour12: false,
    });

    const transaction = `${formatCurrency(amount)}: ${desc || "Không có mô tả"} (${formattedReportDate})`;

    if (type === "thu") {
      income += amount;
      incomeTransactions.push(`+ ${transaction}`);
    } else if (type === "chi") {
      expense += amount;
      expenseTransactions.push(`- ${transaction}`);
    }
  });

  if (!filteredData.length) {
    const range = filter === "week" ? "tuần" : "tháng";
    sendMessage(chatId, `Không có giao dịch cho ${range} được yêu cầu.`);
    return;
  }

  const weekInfo =
    filter === "week"
      ? ` (tuần từ ${now.startOfWeek.toLocaleDateString("vi-VN")} đến ${now.endOfWeek.toLocaleDateString("vi-VN")})`
      : "";

  const report = [
    `Báo cáo (${filter === "all" ? "tổng" : filter}${weekInfo}):`,
    `Tổng thu: ${formatCurrency(income)}`,
    `Tổng chi: ${formatCurrency(expense)}`,
    `Cân đối: ${formatCurrency(income - expense)}`,
    "",
    "Giao dịch thu nhập cụ thể:",
    incomeTransactions.length ? incomeTransactions.join("\n") : "Không có giao dịch thu nhập.",
    "",
    "Giao dịch chi tiêu cụ thể:",
    expenseTransactions.length ? expenseTransactions.join("\n") : "Không có giao dịch chi tiêu.",
  ].join("\n");

  sendMessage(chatId, report);
}
function isValidDate(date, filter, now) {
  if (filter === "month") {
    return (
      date.getMonth() === now.getMonth() &&
      date.getFullYear() === now.getFullYear()
    );
  }
  if (filter === "week") {
    const { startOfWeek, endOfWeek } = now;
    return date >= startOfWeek && date <= endOfWeek;
  }
  return true;
}

function parseDate(filter, dateParam) {
  if (!dateParam) return new Date();
  const parts = dateParam.split("/");
  if (filter === "month" && parts.length === 2) {
    return new Date(parts[1], parts[0] - 1);
  }
  if (filter === "week" && parts.length === 3) {
    const date = new Date(parts[2], parts[1] - 1, parts[0]);
    const dayOfWeek = date.getDay() || 7;
    date.startOfWeek = new Date(date);
    date.startOfWeek.setDate(date.getDate() - dayOfWeek + 1);
    date.endOfWeek = new Date(date.startOfWeek);
    date.endOfWeek.setDate(date.startOfWeek.getDate() + 6);
    return date;
  }
  return new Date();
}

function parseAmount(amount) {
  return parseFloat(amount.replace(/tr/gi, "000000").replace(/k/gi, "000")) || 0;
}

function isValidAmount(amount) {
  return /^[0-9]+(k|tr)?$/i.test(amount);
}

function formatCurrency(amount) {
  return new Intl.NumberFormat("vi-VN", { style: "currency", currency: "VND" }).format(amount);
}

function sendMessage(chatId, text) {
  UrlFetchApp.fetch(`${API_URL}/sendMessage`, {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify({ chat_id: chatId, text, parse_mode: "Markdown" }),
  });
}

```

 ### 2.4 Thay thế:
   - `YOUR_TELEGRAM_BOT_TOKEN` bằng token bot Telegram.
   - `YOUR_SHEET_ID` bằng ID Google Sheets.
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

Truy cập URL sau để kết nối webhook:

```
https://api.telegram.org/bot<TOKEN>/setWebhook?url=<WEB_APP_URL>
```

**Ví dụ:**  
```
https://api.telegram.org/bot123456789:ABCdefGhIJKlmNoPQRstuVWxyZ/setWebhook?url=https://script.google.com/macros/s/AKfycbxEXAMPLE/exec
```

---

---

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
- **Webhook không hoạt động:** Kiểm tra lại TOKEN và URL, Lúc nhấn triển khai đã cấp quyền chưa.
- **Không lưu dữ liệu:** Kiểm tra Sheet ID và quyền truy cập.

***Đóng góp ý tưởng hoặc cần tư vấn liên hệ: t.me/nothing3272***

---
