# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng chờ trả lời bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Vũ Bá Anh  Mã học viên: 2A202602893

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi chuyển service `day12-agent` lên Render, nếu quên khai báo `AGENT_API_KEY`, `Settings` sẽ báo lỗi lúc khởi động thay vì để service nhận request với một khóa mặc định ai cũng biết. Như vậy mình phát hiện cấu hình thiếu trong deploy logs trước khi mở API công khai.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Khi gọi `/ask` qua ứng dụng với Redis giả, stdout ghi đúng một dòng: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T09:37:32.525984+00:00", "user_id": "exercise-log", "tokens_in": 6, "tokens_out": 39, "cost_usd": 2.43e-05}`. Từ các trường này mình lọc request theo `user_id` và cộng `cost_usd`; câu `print("đã trả lời xong")` không cho biết ai gọi hoặc request tốn bao nhiêu.

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
| 1 stage (bản đầu) | Chưa đo; repo không còn bản image 1-stage để so sánh |
| Multi-stage (`k4-l3b-day12-vubaanh-2a202602893-cloudservicesanddeployment-agent:latest`) | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> `docker image ls` đo image multi-stage hiện tại của repo là **271 MB**. Dockerfile cài dependency ở `builder` rồi chép phần cài đặt và source sang `runtime`. Mình không build/đo image 1-stage nên không ghi số so sánh hoặc khẳng định mức chênh lệch.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile chép `requirements.txt` rồi cài dependency trước khi chép `app/` và `utils/`. Sửa một ký tự trong `app/main.py` chỉ làm Docker chạy lại các layer chép source phía sau; layer cài dependency vẫn được lấy từ cache. Nếu `COPY . .` nằm trước `pip install`, sửa source cũng làm layer copy đổi, khiến bước cài dependency chạy lại.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu lỗ hổng trong app cho phép kẻ tấn công chạy lệnh, tiến trình root có thể đọc hoặc sửa các file và tài nguyên mà root trong container được phép truy cập. Nếu tiếp tục thoát khỏi container nhờ một lỗ hổng cấu hình hoặc runtime, quyền cao đó có thể làm tăng ảnh hưởng lên host. `USER appuser` giới hạn quyền của tiến trình ngay từ đầu, nên lỗ hổng app không tự động cấp quyền root trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với giới hạn 10/phút theo phút đồng hồ, có thể gửi **20 request trong 2 giây**: 10 request ở giây 59 của phút này, rồi 10 request ở giây 00 của phút kế tiếp. Sliding window 60 giây vẫn tính cả 20 request đó trong cùng cửa sổ nên chặn phần vượt hạn mức.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong 60 giây; cost guard giới hạn tổng chi phí của một user trong tháng. Một request có thể còn trong hạn 10 request/phút nhưng bị cost guard chặn vì user đã dùng hết ngân sách tháng. Ngược lại, user còn ngân sách tháng nhưng request thứ 11 trong một phút sẽ bị rate limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Khi Redis mất kết nối, nếu cả hai probe đều kiểm tra Redis thì cả ba container lần lượt trả trạng thái lỗi cho probe. Orchestrator hiểu liveness fail là process cần restart và khởi động lại cả ba container; Redis vẫn đang mất kết nối nên probe tiếp tục fail, làm cụm cùng khởi động lại dù bản thân app process có thể còn sống. Tách `/health` khỏi Redis giúp liveness giữ container sống, còn `/ready` báo không nhận traffic cho tới khi Redis hồi phục.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với dict trong RAM, mỗi container giữ một bản lịch sử riêng. Request được gửi tới instance chưa từng thấy user đó có thể trả `history_length: 0`; request kế tiếp tới instance khác cũng không thấy lịch sử của instance trước. Vì vậy con số có thể lặp lại hoặc tăng không đều theo instance. Redis dùng chung giúp mọi instance đọc cùng lịch sử.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần đầu chạy CP5 sau khi chuyển sang Render, bốn test cloud báo lỗi vì `DEPLOYMENT.md` vẫn ghi URL local, nên test không tìm thấy Public URL HTTPS. Mình đọc lỗi fixture `base_url`, thay URL bằng `https://day12-agent-p3pa.onrender.com`, rồi chạy lại: `/health` và `/ready` trả 200, unauthenticated `/ask` trả 401, và authenticated `/ask` cũng thành công. Đây là lỗi hồ sơ deployment thiếu URL thật, không phải app cần thêm route `/`; root 404 vẫn là expected.
