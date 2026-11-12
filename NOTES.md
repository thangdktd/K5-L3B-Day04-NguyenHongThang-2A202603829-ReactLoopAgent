# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Hồng Thắng
MSSV: 2A202603829

## 1. Case được chọn

- ID case: E1
- Câu trả lời của agent: Chuỗi rỗng ("")
- Điểm ở lần chạy 1: 80/100

## 2. Bằng chứng trong trace

trace chứng minh lỗi:
step 1: Tool trả kết quả
step 2: LLM trả finish = "stop"
final answer = ""
status = completed

nguyên nhân có thể xảy ra.
LLM đã gọi tool và nhận dữ liệu nhưng không tạo nội dung trả lời cuối cùng. Trong agent.py, khi nhận finish = "stop", vòng lặp cho phép kết thúc mà chưa kiểm tra câu trả lời có rỗng hay không.
Vì vậy, Agent vẫn báo completed dù answer = "".

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: agent.py, nhánh xử lý if finish == "stop":
Thêm Empty Answer Validation Gate để kiểm tra câu trả lời cuối. Nếu rỗng hoặc chỉ có khoảng trắng, ghi sự kiện vào trace, thêm yêu cầu tổng hợp từ các observation đã có rồi tiếp tục vòng lặp trong giới hạn max_steps.
if finish == "stop":
    answer = message.get("content") or ""
    # CP5: Empty Answer Validation Gate
    if not answer.strip():
        trace.append({"event": "validation_empty_answer", "step": step})
        messages.append({
            "role": "user",
            "content": (
                "Câu trả lời trước đang rỗng. "
                "Hãy tổng hợp các observations đã có "
                "và trả lời rõ ràng bằng tiếng Việt. "
                "Không tự tạo thông tin hoặc nguồn."
            ),
        })
        continue
    # Giữ nguyên các bước kiểm tra và trả kết quả sẵn có bên dưới.
Lý do chọn: Đây là một kiể
- Vì sao chọn thay đổi này: giảm khả năng Agent trả kết quả rỗng mà không cần tăng độ phức tạp của prompt hay can thiệp vào lựa chọn công cụ.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 22 | 22 |
| Điểm case đã chọn | 80/100 | 50/100 |
| Số model calls | 46 | 56 |
| Token, chi phí nếu có | 37.496 | 47.128 |

- Lỗi đã giải quyết: Trong E1, Agent không chấp nhận ngay câu trả lời rỗng mà ghi nhận sự kiện validation và thử trả lời lại
- Lỗi còn lại hoặc mới xuất hiện: Case H4 không được chấm vì Judge báo Điểm boundaries không hợp lệ; Agent vẫn hoàn thành câu trả lời. CP6 vì vậy không có điểm tổng hợp hợp lệ cho toàn bộ 22 case.

## 5. Giới hạn

Điểm Judge: Do LLM chấm nên có thể không chính xác hoặc không nhất quán. Ở CP6, case H4 bị lỗi Judge nên chỉ chấm được 21/22 case.

Snapshot: Dữ liệu từ tool chỉ phản ánh thông tin tại thời điểm lưu, có thể khác với dữ liệu thực tế hiện tại.

Chỉ chạy một lần: Kết quả có thể thay đổi giữa các lần chạy do LLM không hoàn toàn ổn định. Vì vậy, chưa đủ cơ sở để kết luận CP5 tốt hơn CP4.
