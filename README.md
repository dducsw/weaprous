# WeApRous (simple proxy + backend sample)
======================================

Mục đích
--------
Dự án mẫu cho proxy HTTP đơn giản và backend TCP/HTTP (WeApRous). Dùng trong bài tập để hiểu cách định tuyến theo Host và phục vụ file tĩnh / route đơn giản.

Yêu cầu môi trường
------------------
- Python 3.8+ (khuyến nghị)
- Chạy trên Windows PowerShell (hướng dẫn bên dưới)

Các script chính
-----------------
- `start_backend.py`  — khởi backend (mặc định port 9000)
- `start_proxy.py`    — khởi proxy (mặc định port 8080), đọc `config/proxy.conf`
- `start_sampleapp.py`— ví dụ ứng dụng WeApRous (mặc định port 8000)

Chạy nhanh (PowerShell)
-----------------------
Mở PowerShell, vào thư mục dự án:

```powershell
cd 'D:\myBK\HK251\MMT\Assignment1\CO3094-weaprous'
# (tuỳ chọn) tạo và kích hoạt virtualenv
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Chạy backend:

```powershell
python .\start_backend.py --server-ip 0.0.0.0 --server-port 9000
```

Chạy sample app (nếu muốn):

```powershell
python .\start_sampleapp.py --server-ip 0.0.0.0 --server-port 9001
```

Chạy proxy (mở cửa 1 cửa khác):

```powershell
python .\start_proxy.py --server-ip 0.0.0.0 --server-port 8080
```

Kiểm thử nhanh bằng curl/Invoke-WebRequest (gửi header Host tuỳ ý):

```powershell
# dùng curl nếu có
curl -H "Host: 192.168.56.103:8080" http://127.0.0.1:8080/

# hoặc PowerShell Invoke-WebRequest
Invoke-WebRequest -Uri http://127.0.0.1:8080/ -Headers @{ 'Host' = '192.168.56.103:8080' }
```

Ghi chú quan trọng
------------------
- Một số module trong `daemon/` còn TODO; repo hiện chưa hoàn toàn sẵn sàng production. Bạn cần sửa một số chỗ nếu muốn chạy đầy đủ (ví dụ: xử lý threading, đọc file static, xử lý routing đa `proxy_pass`).
- Nếu các host trong `config/proxy.conf` là IP nội bộ (ví dụ `192.168.56.103`), bạn có thể sửa tạm sang `127.0.0.1` để test trên máy local.

Tiếp theo nên làm
-----------------
- Hoàn thiện đa luồng trong `daemon/proxy.py` và `daemon/backend.py`.
- Implement `Response.build_content()` và `build_response_header()` để phục vụ file tĩnh.
- Thêm scripts tiện dụng (`run_backend.ps1`, `run_proxy.ps1`) và tests smoke.

Liên hệ
-------
Nếu cần, tôi có thể áp các sửa tối thiểu để chạy (threads + serving static) và thực hiện một vòng smoke-test; hoặc hướng dẫn chi tiết từng thay đổi để bạn và đồng đội thực hiện.
