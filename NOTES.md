# Ghi chú phân tích · Day 04

- **Họ và tên** DANG LONG NHAT
- **MSSV** 2A202603483

## 1. Case được chọn

A5 yêu cầu liệt kê đầy đủ các incident GitHub Actions trong năm 2025 và đánh giá độ đầy đủ của dữ liệu. Lượt trước cải tiến đạt **45/100**.

> Có tổng cộng 23 sự cố đã ghi nhận, nhưng không có bất kỳ sự cố nào xảy ra trong năm 2025.

## 2. Bằng chứng trong trace

- Bước 1 gọi `get_status` với `provider=github` và `component=Actions`. Snapshot ghi ngày 09/11/2026. Trường `matching_incidents_in_last_50` là 23, còn danh sách `incidents` trả về 5 mục.
- Bước 2 gọi `get_incident_details` cho cả 5 mục. Tất cả bắt đầu vào ngày 04–05/11/2026. Cả 5 URL từ tool xuất hiện trong câu trả lời.
- `grade.mechanical.violations` rỗng. Agent đã dùng dữ liệu tìm kiếm giới hạn để kết luận về cả năm 2025. Phần ghi chú của câu trả lời có nhắc giới hạn 50 incident gần nhất, nhưng câu kết luận vẫn vượt phạm vi đã kiểm tra.

## 3. Thay đổi đã thực hiện

- Bổ sung quy tắc trong `SYSTEM_PROMPT` để đối chiếu khoảng thời gian được hỏi với các mốc trong tool result. Khi dữ liệu thiếu độ bao phủ, agent cần nêu phần đã kiểm tra và giới hạn của kết luận.
- Trong `run_agent`, lưu URL từ các lần `get_incident_details` thành công. Khi câu trả lời thiếu mọi URL đã quan sát, ghi `validation_missing_source` và yêu cầu model sửa trong số lượt còn lại. Nếu hết lượt, hàm trả `max_steps`.

Bản `agent.py` ở commit `e098c8f` đạt 37/37 unit test offline.

## 4. So sánh hai lần chạy

Hai báo cáo dùng cùng 22 case `core` trên snapshot. Agent và judge đều dùng `openai` với `gpt-4o-mini` và giới hạn `max_steps = 8` cho agent.

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Case được chấm | 22/22 | 22/22 |
| Điểm tổng | 77,05/100 | 80,95/100 |
| Điểm A5 | 45/100 | 40/100 |
| Lượt gọi agent | 51 | 51 |
| Lượt gọi agent riêng A5 | 3 | 3 |
| Lượt gọi judge | 22 | 22 |
| Token agent | 44.275 | 51.328 |
| Token judge | 42.256 | 41.717 |
| Tổng token | 86.531 | 93.045 |

Điểm tổng tăng 3,90 điểm và A5 giảm 5 điểm. Tổng token tăng 6.514.

- **Sửa nguồn trong lượt sau.** A7 ghi `validation_missing_source` ở bước 3 và hoàn tất ở bước 4. H6 ghi event ở bước 2 và hoàn tất ở bước 3. Cả hai câu trả lời cuối chứa URL incident đã quan sát từ tool.
- **Lỗi A5 còn lại.** Lượt sau vẫn mở đầu bằng khẳng định “Hiện tại, không có sự cố nào liên quan đến GitHub Actions trong năm 2025.” Trace tiếp tục chỉ trả về năm incident tháng 11/2026. Quy tắc bổ sung vào prompt chưa ngăn được kết luận vượt phạm vi trong lượt đo này.
- **Lỗi ở các case khác.** H1 và H6 vẫn bị giới hạn điểm vì chỉ gọi tool trong một bước model. Cả hai case yêu cầu ít nhất hai bước có tool call. H2 ở lượt sau bị giới hạn 50 điểm vì thiếu lần gọi `get_status`.

## 5. Giới hạn

Tra cứu cả một năm cần nguồn có phạm vi rộng hơn snapshot. Kiểm tra URL chỉ xác nhận sự hiện diện của một URL đã quan sát. Việc đánh giá từng kết luận cần đối chiếu nội dung nguồn.

Điểm LLM judge là ước lượng và có thể dao động. Hai báo cáo ghi một lượt đo cho mỗi phiên bản. Cần nhiều lượt đo để đánh giá độ ổn định và tách tác động của quy tắc trong prompt khỏi tác động của kiểm tra URL.
