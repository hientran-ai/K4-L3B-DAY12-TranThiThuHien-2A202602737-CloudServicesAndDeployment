# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng giữ chỗ bên dưới bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Thị Thu Hiền  Mã học viên: 2A202602737

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ khi deploy bản production, nếu tôi quên cấu hình `AGENT_API_KEY` thì
> ứng dụng sẽ dừng ngay và log báo lỗi validation. Nhờ vậy bản API không thể
> lên mạng trong trạng thái dùng khóa mặc định mà ai cũng đoán được. Nếu để
> `"changeme"`, health check vẫn có thể xanh và người ngoài có thể gọi `/ask`
> bằng khóa đó, làm tiêu quota trước khi tôi phát hiện cấu hình sai.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi thu được khi chạy test `/ask` là:
> `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T15:04:25.478613+00:00", "user_id": "2A202602737", "tokens_in": 5, "tokens_out": 37, "cost_usd": 2.295e-05}`.
> Vì đây là JSON có trường rõ ràng, tôi có thể lọc toàn bộ request của
> `user_id=2A202602737` để điều tra lỗi và cộng `cost_usd` để theo dõi/cảnh báo chi
> phí. Chuỗi `print("đã trả lời xong")` không chứa dữ liệu có cấu trúc để thực
> hiện hai việc đó.

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
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> *Câu trả lời của bạn*

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> *Câu trả lời của bạn*

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng cho phép thực thi lệnh, kẻ tấn công có thể chạy
> lệnh trong container. Khi process đang là root, họ có toàn quyền trong
> container; nếu runtime còn có lỗi escape hoặc host gắn socket/thư mục nhạy
> cảm vào container thì quyền đó có thể được dùng để chiếm host. Lệnh
> `USER app` khiến code ứng dụng chạy bằng user ít quyền, nên ngay ở bước đầu
> kẻ tấn công chỉ nhận được quyền của `app`, không thể tùy ý sửa file hệ thống
> hay dùng các tài nguyên chỉ dành cho root.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 request trong 2 giây: gửi 10 request ở cuối
> phút, ví dụ `10:00:59`, rồi ngay sau khi bộ đếm reset gửi thêm 10 request ở
> `10:01:00`. Cả hai phút lịch đều không vượt mức 10, nhưng thực tế 20 request
> dồn sát nhau. Sliding window nhìn lại đúng 60 giây gần nhất nên ngăn được
> cách lợi dụng ranh giới phút này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tần suất/số request trong một cửa sổ ngắn, còn cost guard
> giới hạn tổng số tiền của từng user trong cả tháng. Một user gửi ít request
> nhưng mỗi request có prompt rất dài có thể vẫn qua rate limit nhưng bị cost
> guard chặn vì hết ngân sách. Ngược lại, user còn nguyên ngân sách nhưng gửi
> hơn 10 request liên tiếp trong một phút sẽ bị rate limit trả 429 dù cost
> guard vẫn cho phép về mặt tiền.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Khi Redis mất kết nối, endpoint đã gộp bắt đầu trả lỗi cho cả readiness lẫn
> liveness của cả 3 container. Load balancer trước tiên loại chúng khỏi danh
> sách nhận traffic; sau đó orchestrator hiểu nhầm cả 3 process đã chết và
> restart chúng. Các container mới vẫn không kết nối được Redis nên tiếp tục
> fail health check và rơi vào vòng lặp restart. Khi Redis phục hồi sau 30
> giây, cụm vẫn cần chờ container khởi động và probe xanh lại, làm thời gian
> gián đoạn dài hơn. Tách `/health` khỏi Redis giữ process sống, còn `/ready`
> chỉ tạm ngừng traffic cho tới khi dependency phục hồi.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> *Câu trả lời của bạn*

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn*
