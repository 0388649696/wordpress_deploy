# Ứng dụng WP vào Docker để xây dựng website cùng với Mariadb kết nối Cloudflare

## A. Wordpress
Kịch bản: Máy đã có dự án cũ, tạo mới sao cho độc lập tránh chồng chéo.
## 1. Chuẩn bị:
### Dự án mới: 
<img width="1104" height="319" alt="image" src="https://github.com/user-attachments/assets/8214af6f-3808-40b9-a98f-f6f69a8d7120" />
Hành động thoát dự án cũ, về với /home trước khi tạo dự án mới.
 + Cấu hình docker compose.yml.
 + Chạy docker compose up -d => kéo các Image về và tạo Container
 + Check: docker ps.
   
### Cấu hình subdomain mới:
+ Cloudflare Dashboard >...> Tunnel
+ Chọn Tunnel của miền mẹ. Edit
+ Thêm subdomain, Service URL, Path
Trọng tâm:
+ kiểm tra cổng ss -tulpn | grep <port_cua_wordpress> nếu bị trùng và đổi.
+ **Service URL** đặt IP của ens33/eth0 + cổng ví dụ: 192.168.222.111:8882
<img width="1536" height="646" alt="image" src="https://github.com/user-attachments/assets/159d1eb9-7701-4134-a571-7b7d836f1dcb" />
 Bởi vì ban đầu Cloudflare Tunnel của ta đang chạy bên trong Docker nên không thể dùng ip 127.0.0.1 thì CF sẽ tìm chính nó thay vì tìm ra ngoài để thấy 8882. Dùng chính IP máy ảo để Container Tunnel "nhìn thấy" dịch vụ đang mở ở cổng 8882 của máy chủ.

### Thiết lập Wordpress: 
Đi thiết lập Wp:
<img width="973" height="478" alt="image" src="https://github.com/user-attachments/assets/cd525636-5e43-490f-93a8-3ff37ac79a76" />
.
<img width="1862" height="957" alt="image" src="https://github.com/user-attachments/assets/27364719-829d-4346-9ae4-d83a5b396f05" />
<img width="1887" height="922" alt="image" src="https://github.com/user-attachments/assets/ce69afcf-e919-40a6-8a86-3b06c49c5fc9" />


### ĐIỂM ĐỘT PHÁ:

| Tiêu chí | Nhận xét |
|---|---|
| Công sức | Rất thấp. Việc dùng Docker giúp triển khai 3 dịch vụ cùng lúc chỉ với 1 lệnh docker compose up. |
| Độ khó | Dễ sử dụng giao diện web, nhưng đòi hỏi kiến thức về Network (như lỗi 502 bạn vừa sửa) để kết nối Cloudflare Tunnel. |
| Tài nguyên | WordPress chạy trên PHP khá ngốn RAM (khoảng 200-400MB cho 1 dự án). Nếu chạy nhiều site trên 1 máy ảo yếu, RAM sẽ bị quá tải (Swapping). |
| Tính tiện dụng | Rất cao cho việc tạo website nhanh, nhưng tốn công tối ưu bảo mật và tốc độ hơn so với code tay thuần túy. |

## B. N8N 
### 1. Tạo và cài n8n
<img width="662" height="102" alt="Screenshot 2026-05-25 152227" src="https://github.com/user-attachments/assets/e1aef58a-4421-4c59-86a2-2441fa1ca710" />
 N8n yêu cầu ssl và https. Để không cần cài ssl. Có phương pháp cấu hình như sau:
```
n8n_wpanhtu:
    image: n8nio/n8n:latest
    restart: always
    container_name: n8n_wpanhtu
    ports:
      - "5678:5678"
    environment:
      - TZ=Asia/Ho_Chi_Minh
      - WEBHOOK_URL=https://n8nanhtu.divu.click/
    volumes:
      - ./n8n_data:/home/node/.n8n
```
 - Mấu chốt ở - WEBHOOK_URL=https://n8nanhtu.divu.click/. Và Tunnel 
 <img width="1406" height="717" alt="image" src="https://github.com/user-attachments/assets/41e4a001-da28-45cc-b0ee-91683206319c" />
. Rồi tạo tk:
<img width="657" height="844" alt="image" src="https://github.com/user-attachments/assets/9b36cc60-fe18-4fec-aeee-ddbe4a3387f9" />


### Điểm quan trọng:
Trong quá trình cấu hình gặp lỗi "Bad lock file is ignored: ./.docker-compose.yml.swp", đồng thời gây ra lỗi "Error establishing a database connection (Wordpress)" và n8n không truy cập được. Đây là cách xử lý:
Nguyên nhân Ổ đĩa đầy (100%) 
   │
   ├──► Ubuntu không ghi được File tạm (.swp) ──► Docker Compose kẹt cú pháp
   │
   ├──► n8n Container ghi đè File Config lỗi ──► File config hỏng (Invalid JSON) ──► n8n Crash liên tục
   │
   └──► MariaDB không tạo được File Lock ────► Container sập (Exit code 1) ────► WordPress mất kết nối (Database Error)
+ Xử lý:
1. Nâng dung lượng lên.
2. Xóa cấu hình và thư mục n8n cũ
<img width="695" height="381" alt="image" src="https://github.com/user-attachments/assets/9800e6e4-f7ed-4bc9-9bca-8623d5636379" />


### 2. Tạo workflow:
- Sau khi bấm vào "Sự kiện trên ứng dụng", một ô tìm kiếm sẽ hiện ra. Bạn gõ chữ Telegram và chọn Telegram Trigger.
- Ở bảng cấu hình bên phải hiện ra tiếp theo, tại mục Event (Sự kiện), bạn chọn là On Message (Khi có tin nhắn đến).
- Tại mục Credential, bấm vào nút **Set up credential**, chọn Create New Credential rồi dán chuỗi Access Token mà lấy từ @BotFather vào.
<img width="570" height="640" alt="image" src="https://github.com/user-attachments/assets/bde86eec-741f-4bfd-a1b8-adbedc9f640b" />
<img width="1342" height="695" alt="image" src="https://github.com/user-attachments/assets/c4d77dd7-16ab-4bb0-8a81-0306ea449db4" />

```
[Telegram Trigger] ──► [Google Gemini] ──► [Code JavaScript] ──► [WordPress Node]
 (Nhận tin nhắn)        (Sinh bài viết JSON)   (Lọc & làm sạch dữ liệu)   (Đăng bài Publish)
```
Thêm node Gemini:
<img width="1852" height="805" alt="image" src="https://github.com/user-attachments/assets/ff5da6c2-7ef9-45ea-bfe1-bedb5cb5ec19" />.
- truy cập vào trang web: https://aistudio.google.com/ lấy key
- Credential, bạn bấm vào ô lựa chọn và chọn Create New Credential. Điền Key.
- Chọn Model
- Prompt: {{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.
- Bật Output Content as JSON
- Bấm Execute step



