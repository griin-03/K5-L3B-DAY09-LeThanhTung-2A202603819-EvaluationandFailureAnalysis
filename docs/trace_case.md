# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (Lê Thanh Tùng - 2A202603819)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID: `E-seeded_regression-20261118-fa3a` |
| Triệu chứng | Grader Lab 8 fail: `status resolved != denied`. Grader `refund_amount_correct` fail (ledger=20000, label=0). Reply thông báo hoàn tiền 20.000đ trong khi khách hàng không đủ điều kiện bồi thường trễ. |
| Bước sai đầu tiên | Bước 3: kind=`model_override` + payload=`{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Bước 1 (`handoff`) và Bước 2 (`investigator`: `lookup_order`) hoàn toàn đúng. Ở bước 2, DB trả về thực tế `delay_days=1` (đơn O-202 chỉ trễ 1 ngày, theo chính sách cần trễ ≥3 ngày mới được bồi thường). Tuy nhiên sang bước 3, agent đã tin theo lời khiếu nại trong ticket của khách ("Đơn giao trễ 5 ngày") và phát sinh event `model_override` đè số ngày trễ từ 1 thành 5 ngày. Dẫn tới bước 4 (`quote`) tính sai quyền lợi và bước 6 phát lệnh hoàn tiền trái phép. |
| Failure mode | **FM-03** (`ticket_trust` — tin ticket hơn dữ liệu DB) kết hợp dẫn tới **FM-02** (`ignore_denied`). |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: Bước 2 đọc DB ra `delay_days=1`. Không hề có bước 3 `model_override`. Bước 3 chuyển thẳng sang `quote` với `amount_vnd: 0, reason: "no_entitlement"`. Kết quả `status: denied`, hoàn tiền 0đ, trùng khớp 100% với ground truth. |
| Regression test | Grader `DbFactKept` kiểm tra không được tồn tại event `model_override` trong trace; test `tests/test_lab9_graders.py::test_override_event_breaks_db_fact` bắt được lỗi này. Trên shape `seeded_regression`, metric `db_fact_kept` rớt xuống 0.9286 (fail ngưỡng 0.95); trên shape `fixed`, metric đạt 1.0 (pass tuyệt đối). |
| Chưa chắc chắn | Chưa thể xác định mô hình có bị ảnh hưởng bởi kỹ thuật prompt injection tinh vi hơn không nếu ticket chứa các chỉ dẫn phức tạp hơn thay vì câu khiếu nại thông thường. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
