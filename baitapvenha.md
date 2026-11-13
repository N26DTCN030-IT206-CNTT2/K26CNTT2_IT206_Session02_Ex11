# BÀI 11: CHUỖI PIPELINE TRÍCH XUẤT ĐÁNH GIÁ NHÀ CUNG CẤP DẠNG JSON SCHEMA
> **Mục tiêu:** Xây dựng Master Prompt trích xuất dữ liệu bán cấu trúc từ văn bản thành chuỗi JSON thô chuẩn xác, phục vụ hệ thống tự động hóa (RPA/ETL).

---

## 1.  Biên Bản Đàm Phán (Dữ Liệu Đầu Vào Giả Lập)

Dưới đây là đoạn văn bản giả lập ghi lại biên bản đàm phán giữa TechShop và nhà phân phối **Viễn Thông Tân Phát** để dùng làm dữ liệu Test:

> **Biên bản đàm phán mua hàng:**
> "Vào ngày 15/11/2026, phòng Mua hàng TechShop đã tiến hành đàm phán với đại diện của nhà phân phối Viễn Thông Tân Phát. Sau khi thảo luận, đối tác đồng ý cung cấp mức tỷ lệ chiết khấu ưu đãi là 12.5% cho toàn bộ các đơn hàng. Về chính sách công nợ, thời hạn thanh toán được chốt là 30 ngày kể từ ngày xuất hóa đơn. Đối tác xác nhận có hỗ trợ đổi trả trực tiếp (RMA direct) tại trung tâm bảo hành của hãng đối với các sản phẩm phát sinh lỗi kỹ thuật. Các ngành hàng mà Viễn Thông Tân Phát sẽ cung ứng chủ lực cho TechShop bao gồm: Router Wifi, Switch, và Cáp quang. Xét về năng lực tài chính và lịch sử giao hàng, bộ phận kiểm soát nội bộ đánh giá rủi ro của nhà cung cấp này ở mức Low (Thấp)."

---

## 2.  Master Prompt Trích Xuất Dữ Liệu

*Prompt dưới đây sử dụng cấu trúc CREATE, ứng dụng Delimiters `<van_ban>` và thiết lập các Constraints cực kỳ nghiêm ngặt để ép AI xuất JSON thô, không sinh ra lỗi Parse.*

```text
[Role]
Bạn là một AI Data Engineer chuyên nghiệp, đóng vai trò như một bộ phân tích cú pháp (Parser) cho hệ thống tự động hóa RPA/ETL của TechShop.

[Task]
Nhiệm vụ của bạn là đọc biên bản đàm phán được đặt trong cặp thẻ delimiters <van_ban>...</van_ban>, trích xuất các thông tin cần thiết và xuất ra DUY NHẤT một chuỗi JSON thô (Raw JSON String).

[Constraints - Bắt buộc tuân thủ 100%]
1. Định dạng đầu ra:
   - TUYỆT ĐỐI KHÔNG sử dụng Markdown Code Block (như ```json hay ```). Chỉ trả về chuỗi JSON bắt đầu bằng { và kết thúc bằng }.
   - TUYỆT ĐỐI KHÔNG thêm bất kỳ câu chào hỏi, giải thích hay văn bản nào khác ngoài JSON.
   - Tránh mọi lỗi cú pháp: Không để lại dấu phẩy thừa ở phần tử cuối, đóng mở ngoặc đầy đủ.
2. Quy tắc Schema & Kiểu dữ liệu:
   - Cấu trúc khóa (Key) phải giống 100% với JSON Schema yêu cầu.
   - Ép đúng Data Type: "discount_rate" phải là Float (chỉ lấy số, bỏ ký hiệu %), "payment_term_days" phải là Integer, "supports_rma_direct" phải là Boolean (true/false).
3. Xử lý ngoại lệ (Fallback):
   - Nếu một trường thông tin không có trong văn bản, hãy gán giá trị là null đối với String/Float/Integer/Boolean, và [] đối với Array.

[JSON Schema]
{
  "vendor_name": "String (Tên nhà cung cấp)",
  "discount_rate": "Float (Tỷ lệ chiết khấu)",
  "payment_term_days": "Integer (Số ngày công nợ)",
  "supports_rma_direct": "Boolean (Có hỗ trợ đổi trả trực tiếp không: true/false)",
  "core_product_lines": "Array of Strings (Danh sách các ngành hàng cung ứng)",
  "risk_assessment": "String (Đánh giá rủi ro: Low/Medium/High)"
}

[Data Input]
<van_ban>
Vào ngày 15/11/2026, phòng Mua hàng TechShop đã tiến hành đàm phán với đại diện của nhà phân phối Viễn Thông Tân Phát. Sau khi thảo luận, đối tác đồng ý cung cấp mức tỷ lệ chiết khấu ưu đãi là 12.5% cho toàn bộ các đơn hàng. Về chính sách công nợ, thời hạn thanh toán được chốt là 30 ngày kể từ ngày xuất hóa đơn. Đối tác xác nhận có hỗ trợ đổi trả trực tiếp (RMA direct) tại trung tâm bảo hành của hãng đối với các sản phẩm phát sinh lỗi kỹ thuật. Các ngành hàng mà Viễn Thông Tân Phát sẽ cung ứng chủ lực cho TechShop bao gồm: Router Wifi, Switch, và Cáp quang. Xét về năng lực tài chính và lịch sử giao hàng, bộ phận kiểm soát nội bộ đánh giá rủi ro của nhà cung cấp này ở mức Low (Thấp).
</van_ban>
```
