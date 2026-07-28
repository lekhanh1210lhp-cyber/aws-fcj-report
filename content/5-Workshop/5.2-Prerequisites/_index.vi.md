---
title: "Điều kiện tiên quyết"
date: "2026-07-28"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Điều kiện tiên quyết

## AWS và môi trường local

Chuẩn bị tài khoản AWS và chọn **Asia Pacific (Singapore), `ap-southeast-1`**. Danh tính sử dụng cần quyền tạo hoặc xem VPC, EC2, RDS, IAM Role và CloudWatch. Luôn áp dụng least privilege khi workshop có phương án cấp quyền hẹp hơn.

Cài Git, VS Code, Python, Node.js/npm và PlatformIO trên Windows. Máy EC2 cần Python 3, `pip`, Git và PostgreSQL client.

Kiểm tra công cụ local:

```powershell
git --version
python --version
node --version
npm --version
pio --version
```

## Phần cứng và kiến thức nền

Chuẩn bị YOLO UNO, DHT20, cảm biến ánh sáng, quạt, relay hoặc đèn, servo rèm, dây nối và nguồn phù hợp. Bạn nên biết REST request cơ bản, PostgreSQL, Security Group, biến môi trường và Linux `systemd`.

## Checklist

- [ ] AWS Console đang ở `ap-southeast-1`.
- [ ] Các lệnh local trả về phiên bản.
- [ ] Có đủ phần cứng và dây nối.
- [ ] Có thể clone source repository.
- [ ] Không lưu credential trong file được Git theo dõi.

**Kết quả mong đợi:** Nhóm có thể bắt đầu tạo hạ tầng AWS mà không bị gián đoạn do thiếu công cụ hoặc quyền truy cập.

## Xử lý sự cố

- Nếu hệ thống không nhận `python` hoặc `pio`, mở lại terminal sau khi cài và kiểm tra `PATH`.
- Nếu AWS báo từ chối quyền, ghi lại tên action bị deny và yêu cầu quản trị viên cấp đúng quyền tối thiểu.

Tiếp theo: [xem kiến trúc và các luồng dữ liệu](../5.3-Architecture-and-Service-Design/).
