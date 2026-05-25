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

