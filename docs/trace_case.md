# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

## Bảng phân tích chi tiết

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID `R-seeded_regression-C-002-11` |
| Triệu chứng | `TrialPass` (lab8 grader) fail (`status resolved != denied`), `DbFactKept` fail (`1 override event(s)`), `RefundAmountCorrect` fail (`ledger refund_vnd=20000 != expected 0`). Reply thông báo đã hoàn 20.000đ trong khi sổ cái đáng lẽ phải từ chối hoàn (0đ). |
| Bước sai đầu tiên | Event 3: `model_override` `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) và Event 2 (`investigator`) hoàn toàn chính xác: investigator tra cứu đúng đơn O-202 với `delay_days = 0`. Tuy nhiên ngay sau đó tại Event 3, agent đã tin vào văn bản ticket ("Đơn giao trễ 5 ngày") và chèn event `model_override` đè lên sự thật của DB. Mọi bước sau đó (Event 4 quote 20.000đ, Event 6 hoàn tiền và reply) chỉ là hệ quả sai dây chuyền từ Event 3. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Cùng claim `C-002@11` trên shape `fixed`: Không có event `model_override`. Event 3 chuyển thẳng sang `quote` với `amount_vnd: 0, reason: "no_entitlement"`, ledger ghi nhận 0đ, trạng thái `denied`, khớp hoàn toàn với nhãn chuẩn. |
| Regression test | `test_override_event_breaks_db_fact` trong `tests/test_lab9_graders.py` sử dụng grader `DbFactKept()`. Test này bắt được lỗi ngay tại bước trace có chứa event `model_override`. Grader fail trên variant `seeded_regression` và pass 100% trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận payload `delay_days: 5` bị override; trace không giải thích được vì sao mô hình lại quyết định tin vào lời khai trong ticket thay vì ưu tiên dữ liệu DB (cần xem xét prompt và hướng dẫn system prompt của agent). |

## Quy tắc
- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: `DbFactKept` đã được tích hợp vào bộ grader chính thức.
