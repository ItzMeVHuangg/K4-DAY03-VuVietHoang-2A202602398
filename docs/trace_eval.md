# 📊 BÁO CÁO THU HOẠCH NGHIỆM THU BÀI LAB 3 (BƯỚC 3 — SUBMISSION ARTIFACT)

> **Họ và Tên Học viên:** Vũ Việt Hoàng 
> **Mã Sinh Viên / Mã Học viên:** 2A202602398  
> **Chủ đề Lựa chọn:** Gợi ý 1.1 — Trợ lý Học vụ & Tra cứu Lịch thi VinUni (tra cứu điểm GPA, lịch thi và đặt lịch tư vấn học vụ với Cố vấn)  

---

## 1. BẢNG CHẤM ĐIỂM AGENTIC FIT SCORING MATRIX (ĐÁNH GIÁ CHỦ ĐỀ)

| Tiêu chí Đánh giá | Mức độ (1 - 5) | Giải trình chi tiết lý do chọn điểm |
| :--- | :---: | :--- |
| **1. Multi-step Reasoning** | 4 / 5 | Nhiều yêu cầu phải tách thành chuỗi bước nối tiếp. Ví dụ: *"GPA của SV2026001 thế nào, nếu thấp thì đặt lịch gặp cố vấn chiều 15/09"* → (1) tra cứu hồ sơ học vụ, (2) đánh giá GPA, (3) lấy tên cố vấn phụ trách từ hồ sơ, (4) đặt lịch hẹn. Không chấm 5 vì phần lớn luồng chỉ dài 2–3 bước, không cần lập kế hoạch phức tạp. |
| **2. Tool Interaction** | 5 / 5 | Toàn bộ dữ liệu cốt lõi (GPA, trạng thái học, cố vấn, lịch thi, lịch hẹn) nằm trong hệ thống học vụ bên ngoài, LLM không thể tự biết. Bắt buộc gọi tool qua MCP Server: `academic_query` để đọc dữ liệu và `schedule_appointment` để **ghi** (tạo booking). Chatbot thuần không có tool sẽ bịa GPA/lịch hẹn → sai nghiệp vụ. |
| **3. Dynamic Decision** | 4 / 5 | Bước tiếp theo phụ thuộc trực tiếp vào Observation: nếu `academic_query` trả `NOT_FOUND` → dừng và hỏi lại mã SV thay vì đặt lịch; nếu tìm thấy → dùng trường `advisor` trong kết quả làm `advisor_name` cho lượt gọi sau; nếu người dùng thiếu thời gian hẹn → hỏi lại trước khi gọi tool. Không chấm 5 vì tập nhánh rẽ còn hữu hạn và dễ dự đoán. |
| **4. Long Horizon Goal** | 3 / 5 | Agent cần giữ mục tiêu "đặt được lịch tư vấn cho đúng sinh viên" qua vài vòng Thought → Action → Observation và có thể qua nhiều lượt hội thoại (bổ sung mã SV, chốt giờ). Tuy vậy mỗi phiên thường kết thúc trong một vài lượt, không cần memory dài hạn hay theo dõi tiến độ nhiều ngày → mức trung bình. |
| **TỔNG ĐIỂM AGENTIC FIT** | **16 / 20** | *16 > 12 → Bài toán **phù hợp** triển khai ReAct Agent (Cấp 3) kết nối MCP Server; chưa cần Autonomous Agent (Cấp 4) vì mục tiêu ngắn hạn và phạm vi tool nhỏ.* |

---

## 2. TRÍCH XUẤT KẾT QUẢ WATERFALL TRACE LOG (SAU KHI CHẠY TEST SUITE TRÊN API THẬT)

> ⚠️ **YÊU CẦU NGHIỆM THU:** Mở tệp `.env` điền `GEMINI_API_KEY` (hoặc `OPENAI_API_KEY`) để kết nối LLM thật trước khi thực thi `python src/app.py --all`. Bài nộp chỉ dùng Mock Offline Provider sẽ không đạt điểm nghiệm thực tế.

Dán 1 đoạn trích xuất log tiêu biểu từ file `docs/trace_waterfall.json` sinh ra từ phản hồi LLM API thật:

```json
[
  {
    "step": 1,
    "query": "Mình là SV2026002, mình quên tên cố vấn học tập của mình, kiểm tra giúp mình rồi đặt lịch gặp cố vấn đó lúc 10:00 ngày 20/09/2026",
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'academic_query' với tham số: {\"student_id\": \"SV2026002\"}",
    "tool_name": "academic_query",
    "arguments": {
      "student_id": "SV2026002"
    },
    "observation": {
      "status": "SUCCESS",
      "student_id": "SV2026002",
      "data": {
        "full_name": "Trần Thị Bình",
        "class": "AI-K4",
        "gpa": 3.6,
        "email": "binh.tt@vinuni.edu.vn",
        "status": "Đang học",
        "advisor": "TS. Lê Thị B"
      }
    },
    "latency_ms": 2372.82
  },
  {
    "step": 2,
    "query": "Mình là SV2026002, mình quên tên cố vấn học tập của mình, kiểm tra giúp mình rồi đặt lịch gặp cố vấn đó lúc 10:00 ngày 20/09/2026",
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'schedule_appointment' với tham số: {\"datetime_str\": \"10:00 20/09/2026\", \"student_id\": \"SV2026002\", \"advisor_name\": \"TS. Lê Thị B\"}",
    "tool_name": "schedule_appointment",
    "arguments": {
      "datetime_str": "10:00 20/09/2026",
      "student_id": "SV2026002",
      "advisor_name": "TS. Lê Thị B"
    },
    "observation": {
      "status": "SUCCESS",
      "booking_id": "BK-SV2026002-99",
      "student_id": "SV2026002",
      "datetime": "10:00 20/09/2026",
      "advisor": "TS. Lê Thị B",
      "message": "Đặt lịch thành công cho sinh viên SV2026002 với TS. Lê Thị B vào lúc 10:00 20/09/2026."
    },
    "latency_ms": 2509.21
  },
  {
    "step": 3,
    "query": "Mình là SV2026002, mình quên tên cố vấn học tập của mình, kiểm tra giúp mình rồi đặt lịch gặp cố vấn đó lúc 10:00 ngày 20/09/2026",
    "action_type": "FINAL_ANSWER",
    "thought": "Gemini phản hồi trực tiếp bằng văn bản (không cần gọi công cụ).",
    "output": "Chào bạn, \n\nMình đã kiểm tra thông tin và hỗ trợ đặt lịch hẹn cho bạn thành công:\n\n* **Cố vấn học tập của bạn:** TS. Lê Thị B\n* **Thông tin lịch hẹn:**\n  * **Thời gian:** 10:00 ngày 20/09/2026\n  * **Mã lịch hẹn:** BK-SV2026002-99\n\nNếu cần hỗ trợ thêm bất kỳ thông tin nào khác, bạn cứ tự nhiên hỏi nhé! Chúc bạn có một buổi tư vấn học tập hiệu quả.",
    "latency_ms": 3368.68
  }
]
```

---

## 3. TỔNG KẾT KẾT QUẢ NGHIỆM THU & NỘP BÀI

- [x] Đã điền API Key thật trong `.env` và xác nhận Agent chạy mượt mà trên LLM API thật (Gemini/OpenAI).
- **Tổng số Test Cases đã chạy thành công:** 5 / 5 test cases.
- **Số lượt gọi Tool qua MCP Server chính xác:** 5 lượt.
- **Kết quả đẩy Repo nộp bài:** [ ] Đã Commit và Push mã nguồn thành công lên GitHub cá nhân.

---

> ✅ **HOÀN TẤT NỘP BÀI:** Sao chép đường link GitHub Repository cá nhân của bạn và dán vào ô nộp bài trên hệ thống LMS VLearn để hoàn tất Bài Lab 3!
