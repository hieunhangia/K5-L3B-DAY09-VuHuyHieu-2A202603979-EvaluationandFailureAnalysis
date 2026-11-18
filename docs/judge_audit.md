# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Đạt (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không đạt (κ=0.375 < 0.6, R=0.300 < 0.7) | loại |

## ≥5 case audit thủ công (trích xuất từ các case bất đồng của `judge_helpful`)

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | HL-015 | PASS | FAIL | Người đúng | Câu trả lời "Dạ, LogiX đã hoàn 55.000đ cho đơn O-112 ạ" quá cụt lủn, không giải thích lý do tính ra số tiền hoàn hay hướng dẫn bước tiếp theo cho khách hàng. Judge bị đánh lừa bởi câu trả lời ngắn gọn có vẻ lịch sự. |
| 2 | HL-016 | PASS | FAIL | Người đúng | Câu trả lời chỉ là văn mẫu xin lỗi xã giao ("Dạ, LogiX thành thật xin lỗi anh/chị..."), hoàn toàn không giải quyết thắc mắc hay đưa ra giải pháp hoàn tiền. Judge nhầm lẫn giữa thái độ xin lỗi và tính hữu ích (helpfulness). |
| 3 | HL-020 | PASS | FAIL | Người đúng | Câu trả lời chỉ thông báo đã hoàn tiền mà không cung cấp thời gian tiền về tài khoản, phương thức hoàn hay mã tham chiếu giao dịch. Thiếu thông tin thiết yếu đối với người nhận hoàn tiền. |
| 4 | HL-028 | PASS | FAIL | Người đúng | Tương tự HL-016, câu trả lời xin lỗi lặp lại rập khuôn mà không có bất kỳ hành động giải quyết sự cố đơn hàng nào. Judge cho PASS vì thấy lời xin lỗi lịch thiệp. |
| 5 | HL-035 | PASS | FAIL | Người đúng | Lời xin lỗi chung chung mong khách hàng thông cảm nhưng không giải quyết vấn đề khiếu nại cốt lõi của khách. Người chấm đánh FAIL là chuẩn xác. |

## Kết luận
- **Tiêu chí bị loại và lý do**: 
  - Tiêu chí `helpful` bị loại hoàn toàn khỏi CI Release Gate.
  - Ma trận nhầm lẫn của `helpful`: `{"tp": 9, "fp": 0, "fn": 21, "tn": 70}`. 
  - Tỷ lệ bỏ sót lỗi (False Negative) cực kỳ nghiêm trọng: Judge bỏ lọt tới 21/30 trường hợp không hữu ích (Recall chỉ đạt $30\%$, Kappa chỉ đạt $0.375$). Judge bị đánh lừa bởi các câu trả lời chứa lời xin lỗi lịch sự hoặc câu văn ngắn gọn nhưng rỗng nội dung hỗ trợ.
  - Hướng sửa Rubric cho `helpful`: Bổ sung tiêu chí đánh giá bắt buộc phải có thông tin hành động cụ thể (Actionable Information), phân biệt rõ ràng giữa "lịch sự" (politeness/tone) và "giải quyết được vấn đề" (helpfulness/resolution).
- **Rủi ro còn lại**: 
  - Cả hai judge hiện chỉ được kiểm tra trên dữ liệu replay đã công khai (`judge_tone_public.csv`, `judge_helpful_public.csv`).
  - Cần thu thập tập nhãn người mới (held-out test set) từ các tương tác thực tế của người dùng để tránh overfitting vào tập đánh giá public.
