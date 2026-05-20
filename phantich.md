# BẢNG THIẾT KẾ REST API HỆ THỐNG QUẢN LÝ MƯỢN SÁCH

| STT | Chức năng | Method | URL | Query params / Body | Status thành công | Status lỗi (ví dụ) |
|-----|-----------|--------|-----|---------------------|-------------------|-------------------|
| 1 | Lấy danh sách sách (có phân trang & lọc theo tác giả) | GET | `/api/v1/books` | Query: `?page=1&limit=10&author=...` | 200 OK | 400 Bad Request (sai tham số phân trang) |
| 2 | Lấy thông tin chi tiết một cuốn sách | GET | `/api/v1/books/{id}` | Không có | 200 OK | 404 Not Found (ID sách không tồn tại) |
| 3 | Thêm sách mới vào thư viện | POST | `/api/v1/books` | Body: `{ "title": "...", "author": "...", "year": 2026, "quantity": 5 }` | 201 Created | 400 Bad Request (thiếu trường bắt buộc) |
| 4 | Cập nhật số lượng sách (chỉ chỉnh sửa 1 trường số lượng) | PATCH | `/api/v1/books/{id}` | Body: `{ "quantity": 10 }` | 200 OK | 404 Not Found (ID sách không tồn tại) |
| 5 | Xóa sách khỏi hệ thống | DELETE | `/api/v1/books/{id}` | Không có | 204 No Content | 404 Not Found (ID sách không tồn tại) |
| 6 | Lấy danh sách tất cả thẻ mượn của một cuốn sách cụ thể | GET | `/api/v1/books/{id}/loans` | Không có | 200 OK | 404 Not Found (ID sách không tồn tại) |
| 7 | Tạo thẻ mượn sách mới (Mượn sách) | POST | `/api/v1/loans` | Body: `{ "bookId": "{id}", "borrowerName": "...", "borrowDate": "2026-05-20" }` | 201 Created | 400 Bad Request (Sách đã hết số lượng để mượn) |
| 8 | Trả sách (cập nhật thông tin ngày trả thực tế) | PATCH | `/api/v1/loans/{id}` | Body: `{ "returnDate": "2026-05-30" }` | 200 OK | 404 Not Found (Không tìm thấy ID thẻ mượn này) |

---

## Giải thích tư duy thiết kế hệ thống (Glossary & Kiến trúc)

### 1. Phiên bản hóa (Versioning)

Thêm tiền tố `/api/v1` vào trước tài nguyên để dễ dàng nâng cấp logic hệ thống lên v2, v3 sau này mà không làm sập các ứng dụng Client cũ đang chạy.

### 2. Sub-resource (Tài nguyên phụ thuộc - Dòng số 6)

URL `/books/{id}/loans` thể hiện mối quan hệ sở hữu chặt chẽ: Các thẻ mượn này **thuộc về** cuốn sách có ID cụ thể kia. Thiết kế này chuẩn RESTful hơn việc dùng dạng phẳng như `/loans?bookId=...`

### 3. Sử dụng PATCH (Dòng số 4 và số 8)

- **Đối với hành động thay đổi số lượng sách (dòng 4):** Dùng PATCH vì chỉ cập nhật duy nhất trường `quantity`, giữ nguyên `title`, `author`, `year`. Nếu dùng PUT, Client buộc phải gửi lại toàn bộ thông tin gốc, nếu không các trường còn lại sẽ bị reset về rỗng (null).

- **Đối với hành động trả sách (dòng 8):** Dùng PATCH lên thẻ mượn vì ban đầu khi mượn sách, trường `returnDate` đang để trống hoặc mặc định. Khi trả sách ta chỉ cập nhật bổ sung giá trị cho trường `returnDate` này.