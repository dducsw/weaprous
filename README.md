# WeApRous - Web App Routers

---

## 🧩 1. Tổng quan dự án WeApRous

**WeApRous (WebApp Routes)** là một **framework học tập** được thiết kế cho môn *Computer Network (CO3093/CO3094)* tại HCMUT, giúp sinh viên **hiểu và lập trình toàn bộ hệ thống mạng ứng dụng thực tế** — bao gồm HTTP server, proxy server, backend server, và web application RESTful — **mà không dùng các framework có sẵn như Flask/Django**.

Hệ thống mô phỏng một môi trường **multi-process** gồm:

```
Client  →  Proxy  →  Backend(s)  →  WebApp (WeApRous)
```

> 🎯 Mục tiêu:
>
> * Hiểu mô hình **Client–Server** và **Peer-to-Peer (P2P)**.
> * Tự lập trình tầng **Socket + HTTP**.
> * Xây dựng **RESTful API framework tối giản (WeApRous)**.
> * Thực hành **session bằng cookies** và **ứng dụng chat phân tán.**

---

## ⚙️ 2. Kiến trúc hệ thống

### 🔸 Thành phần chính

| Thành phần             | Mô tả                                                                                                              | Tệp liên quan                                            |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------- |
| **Proxy**              | Trung gian giữa client và backend. Quản lý traffic, định tuyến request đến backend phù hợp, hỗ trợ load balancing. | `start_proxy.py`, `daemon/proxy.py`, `config/proxy.conf` |
| **Backend**            | Xử lý logic chính của ứng dụng, phản hồi request từ proxy.                                                         | `start_backend.py`, `daemon/backend.py`                  |
| **WebApp (WeApRous)**  | Framework RESTful tự viết, cho phép định nghĩa route và phương thức xử lý (GET, POST, PUT, DELETE).                | `daemon/weaprous.py`, `start_sampleapp.py`               |
| **HTTP Adapter**       | Giao tiếp giữa socket server và tầng ứng dụng HTTP (parse, build, gửi/nhận HTTP message).                          | `daemon/httpadapter.py`                                  |
| **Request / Response** | Định nghĩa đối tượng HTTP request và response (header, cookies, MIME type, status code).                           | `daemon/request.py`, `daemon/response.py`                |
| **Dictionary utils**   | Lớp `CaseInsensitiveDict` để xử lý header không phân biệt hoa thường.                                              | `daemon/dictionary.py`                                   |

---

## 🧱 3. Cách hoạt động

### 3.1. Proxy Server (`start_proxy.py`)

* Được chạy bằng lệnh:

  ```bash
  python start_proxy.py --server-ip 0.0.0.0 --server-port 8080
  ```
* Proxy đọc tệp `config/proxy.conf` để nạp danh sách backend mapping:

  ```nginx
  host "app1.local" {
      proxy_pass http://192.168.56.103:9001;
  }
  host "app2.local" {
      proxy_pass http://192.168.56.220:9002;
      dist_policy round-robin;
  }
  ```
* Sau đó gọi hàm `create_proxy(ip, port, routes)` để khởi tạo server lắng nghe.
* Nhiệm vụ chính:

  * Nhận HTTP request từ client.
  * Phân tích header `Host`.
  * Gửi request đến backend tương ứng (qua TCP socket).
  * Nhận response và gửi lại client.

---

### 3.2. Backend Server (`start_backend.py`)

* Chạy bằng:

  ```bash
  python start_backend.py --server-ip 0.0.0.0 --server-port 9000
  ```
* Gọi `create_backend(ip, port)` → khởi động socket server đa luồng.
* Mỗi kết nối được xử lý bởi một `HttpAdapter`:

  * Đọc request.
  * Phân tích HTTP header, cookie.
  * Gọi hàm xử lý tương ứng (nếu có route).
  * Xây dựng và gửi response (dạng HTTP raw bytes).

---

### 3.3. Web Application (WeApRous)

* Là **mini-framework** tương tự Flask, cho phép đăng ký các route RESTful:

  ```python
  from daemon.weaprous import WeApRous
  app = WeApRous()

  @app.route('/login', methods=['POST'])
  def login(headers="guest", body="anonymous"):
      return {"message": "Login success"}

  @app.route('/hello', methods=['GET'])
  def hello(headers, body):
      return {"message": "Hello, world!"}

  app.prepare_address('127.0.0.1', 8000)
  app.run()
  ```
* Mỗi route được ánh xạ vào **path + method**, xử lý bằng socket server nội bộ (TCP), phục vụ REST API thuần túy.

---

## 💬 4. Các bài tập và yêu cầu chính

### 🧩 Task 1A – HTTP Authentication & Cookie Session

* Khi người dùng gửi `POST /login` với `username=admin`, `password=password` → server gửi `Set-Cookie: auth=true`.
* Khi truy cập `/`, nếu cookie tồn tại → hiển thị index.html, ngược lại trả về `401 Unauthorized`.

### 🧩 Task 1B – Cookie-based Access Control

* Duy trì phiên đăng nhập bằng cookie giữa các request.
* Dùng mô hình TCP đa luồng để phục vụ nhiều client đồng thời.

---

### 🧩 Task 2 – Hybrid Chat Application (Client-Server + P2P)

* **Client-Server phase**: peer đăng ký với server (tracker), lấy danh sách các peers khác.
* **P2P phase**: peers kết nối trực tiếp với nhau, gửi tin nhắn qua TCP.
* **RESTful endpoints** ví dụ:

  ```
  PUT /login/
  POST /submit-info/
  GET /get-list/
  POST /connect-peer/
  POST /send-peer/
  ```
* Giao diện hiển thị kênh, tin nhắn, thông báo (notification).

---

## 🧰 5. Quy trình triển khai

| Bước | Công việc                                                | Kết quả                                      |
| ---- | -------------------------------------------------------- | -------------------------------------------- |
| 1    | Cấu hình `proxy.conf`                                    | Định tuyến traffic đến backend               |
| 2    | Chạy backend server (`start_backend.py`)                 | Cung cấp dịch vụ HTTP cơ bản                 |
| 3    | Chạy proxy server (`start_proxy.py`)                     | Giao tiếp với client                         |
| 4    | Chạy webapp (`start_sampleapp.py`)                       | RESTful API hoạt động                        |
| 5    | Kiểm tra truy cập qua trình duyệt tại `http://<IP>:8080` | Ứng dụng hoạt động, chuyển hướng qua backend |

---

## 🧮 6. Công nghệ và kỹ thuật sử dụng

| Công nghệ               | Vai trò                                       |
| ----------------------- | --------------------------------------------- |
| **Python socket**       | Giao tiếp TCP/IP giữa các tiến trình          |
| **threading**           | Xử lý đa kết nối client đồng thời             |
| **argparse**            | Truyền tham số khởi chạy (IP, port)           |
| **re (regex)**          | Phân tích file cấu hình proxy                 |
| **Custom HTTP parser**  | Phân tích và xây dựng HTTP request/response   |
| **CaseInsensitiveDict** | Quản lý header không phân biệt chữ hoa/thường |
| **Cookies (auth=true)** | Duy trì session người dùng                    |

---

## 📊 7. Đánh giá và chấm điểm (theo tài liệu gốc)

| Thành phần                  | Điểm        |
| --------------------------- | ----------- |
| **Cookies (session)**       | 3 điểm      |
| **ChatApp – Client/Server** | 2 điểm      |
| **ChatApp – Peer-to-Peer**  | 2 điểm      |
| **Report**                  | 3 điểm      |
| **Tổng cộng**               | **10 điểm** |

---

## 🧠 8. Ý nghĩa học thuật

WeApRous giúp sinh viên:

* Hiểu **tầng ứng dụng (Application Layer)** trong TCP/IP.
* Thực hành triển khai **HTTP server thủ công** (không framework).
* Hiểu hoạt động **của proxy, backend, cookie, session**.
* Nắm mô hình **RESTful API và socket programming**.
* Tiếp cận thực tế mô hình **multi-tier architecture (Proxy–Backend–WebApp)**.

---

Nếu bạn muốn, mình có thể:

* 🧩 Vẽ **sơ đồ kiến trúc WeApRous** bằng diagram chi tiết.
* ⚙️ Tạo **script mẫu hoàn chỉnh** chạy proxy + backend + webapp.
* 📖 Viết **báo cáo tóm tắt** theo mẫu nộp cho Assignment 1 (10 trang chuẩn).

Bạn muốn mình làm phần nào trước?
