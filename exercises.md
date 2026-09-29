# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: hãy viết câu trả lời trực tiếp dưới mỗi câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Đức Anh Quân  Mã học viên: L3B202602405

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu mình deploy lên Railway mà quên đặt `AGENT_API_KEY`, app sẽ khởi động nhưng không có khóa nào để xác thực. Nếu để mặc định là `"changeme"`, mọi client đều có thể gọi `/ask` mà không cần biết khóa thật, làm lộ chi phí và khiến hầu hết request quay về phía người lạ. Với fail fast, app dừng ngay để mình sửa cấu hình trước khi có ai sử dụng, tránh “sai khóa nhưng vẫn chạy” kéo dài và ăn tiền sai.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log ví dụ: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T11:18:32.451298+00:00", "user_id": "sv-test", "tokens_in": 21, "tokens_out": 18, "cost_usd": 0.0002}`.
>
> Hai việc log JSON giúp được mà `print("đã trả lời xong")` không làm được là: (1) nhóm và filter log theo sự kiện, mức độ, user, hoặc chi phí để theo dõi production; (2) gửi lên hệ thống giám sát/Datadog/Cloud Logging để tự động cảnh báo hoặc thống kê số request, chi phí, lỗi, tần suất 429/402 theo thời gian.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | khoảng 1GB+ |
| Multi-stage | khoảng 300–500MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Sự chênh lệch chủ yếu là các package build-time và toolchain của Python không cần thiết cho runtime: `pip`, `setuptools`, compiler, thư viện dev, cache, và code không chạy hiện tại. Một Dockerfile 1-stage copy nguyên cả source và cài dependencies vào cùng layer, nên image chứa rất nhiều thứ thừa. Multi-stage chỉ copy phần runtime thật sự cần vào image cuối cùng, nên image nhỏ hơn nhiều và tiêu tốn ít bộ nhớ/đĩa hơn khi deploy.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Với Dockerfile đúng thứ tự, `FROM`, `WORKDIR`, `COPY requirements.txt`, `RUN pip install` được cache lại nếu chỉ `app/main.py` thay đổi. Khi mình sửa một dòng trong `app/main.py`, chỉ layer `COPY app ./app` và các layer sau đó phải rebuild; phần cài dependency không cần cài lại. Nếu đặt `COPY . .` lên trước `RUN pip install`, thì mọi thay đổi trong source code đều làm layer copy source đổi, kéo theo layer `pip install` phải chạy lại, gây rebuild chậm và lãng phí thời gian và băng thông khi phát triển.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Ví dụ: code Python có lỗ hổng RCE hoặc file upload, kẻ tấn công khai thác được và chạy một lệnh trong container. Nếu container chạy bằng root, lệnh đó có quyền root bên trong container. Nếu chế độ chạy container cho phép mount volume hoặc có lỗi kernel/container escape, quyền root trong container có thể mở rộng lên quyền trên máy host. `USER appuser` cắt đứt chuỗi đó vì tiến trình bị giới hạn ở user không phải root, không thể tạo file hệ thống quan trọng, không thể sửa các file của host và giảm đáng kể rủi ro nếu bị xâm nhập.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với cách đếm theo phút đồng hồ, người dùng có thể gửi tối đa 20 request trong 2 giây liên tiếp. Ví dụ: lúc 10:00:59 gửi 10 request, rồi minute đổi sang 10:01:00 và reset bộ đếm, tiếp tục gửi 10 request ở 10:01:01. Cách đếm cũ “reset mỗi phút” cho phép vượt qua giới hạn ngay khi cửa sổ thay đổi, trong khi sliding window ngăn chặn kiểu này bằng cách đếm request trong 60 giây liên tục.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số lượng request trong một khoảng thời gian; cost guard giới hạn tổng tiền tiêu trong tháng. Một tình huống rate limit cho qua nhưng cost guard chặn là: 3 request liên tiếp đều rất lớn, mỗi request tốn 0.4 USD, tổng tốn 1.5 USD nhưng vẫn dưới giới hạn 3 request/phút; cost guard sẽ chặn khi tổng cộng vượt ngân sách người dùng. Tình huống ngược lại là: nhiều request rất nhỏ, mỗi request chỉ tốn 0.001 USD, nhưng người dùng spam rất nhanh; rate limit sẽ chặn vì số request vượt quá 10/phút dù chi phí vẫn ở mức thấp.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Thứ tự sẽ như sau: Redis mất kết nối; `/health` và `/ready` bắt đầu trả lỗi hoặc 503 nếu cùng check Redis; load balancer nhìn thấy instance không sẵn sàng và ngừng gửi traffic mới vào nó; các request đang đang chạy có thể tiếp tục đến hết, nhưng không có request mới được gửi; sau 30 giây, cuối cùng cụm sẽ có ít nhất một container được đánh dấu không sẵn sàng, và traffic chuyển sang các instance còn sống. Nếu `/health` vẫn nhẹ và không phụ thuộc Redis, then traffic mới không bị chặn chỉ vì Redis chết tạm thời; đó là lý do `/health` và `/ready` nên tách riêng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu history nằm trong một dict Python trên RAM của từng container, mỗi instance sẽ có bộ nhớ riêng. Khi scale lên 3 agent, request đến instance A sẽ thấy lịch sử 1, request sang instance B có thể thấy 0 hoặc khác hoàn toàn vì dữ liệu không chia sẻ. `history_length` sẽ không ổn định giữa các request cùng user_id, và khi restart container nó sẽ mất sạch lịch sử. Khi dùng Redis, tất cả instance cùng nhìn thấy cùng một key `history:user`, nên `history_length` tăng tuần tự và nhất quán giữa các request.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Một lỗi phổ biến là `REDIS_URL` sai: trên cloud, `localhost` không phải Redis server, mà là chính container app. Tôi đã gặp tình huống `ready` trả 503 hoặc `/health` vẫn sống nhưng `/ready` báo không kết nối được Redis, vì biến môi trường đặt là `redis://localhost:6379/0` thay vì `redis://<host>:6379/0` của add-on Redis trên platform. Tôi đã kiểm tra log và endpoint `/ready`, rồi sửa biến `REDIS_URL` sang đúng giá trị add-on, khởi động lại service, và ngay sau đó `ready` trả `200 {"status":"ready"}`.
