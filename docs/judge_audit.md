# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Đạt (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không đạt (κ=0.375 < 0.6, R=0.300 < 0.7) | loại |

## ≥5 case audit thủ công (từ tập `judge_tone_public.csv`)
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người | Reply xưng hô quá thân mật, thiếu trang trọng ("Em đã hoàn tiền cho anh rồi nha, anh check giúp em."), người dán nhãn FAIL là chuẩn xác theo chuẩn dịch vụ CSKH LogiX. Judge quá lỏng tay khi bỏ qua (FN). |
| 2 | H-017 | PASS | FAIL | Người | Xưng hô "bạn ơi... nha" thiếu chuẩn mực chuyên nghiệp doanh nghiệp. Judge phân loại PASS là lỏng (FN). |
| 3 | H-029 | PASS | FAIL | Người | Giọng điệu cộc lốc, mệnh lệnh ("không cần nhắn thêm"), gây cảm giác tiêu cực/thô lỗ cho khách. Người gắn nhãn FAIL là đúng, Judge không phát hiện được sắc thái này (FN). |
| 4 | H-042 | FAIL | PASS | Người | Reply mẫu mực, đầy đủ thông tin ("Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."). Judge bắt lỗi nhầm (FP) có thể do diễn đạt mốc thời gian "3-5 ngày". |
| 5 | H-051 | FAIL | PASS | Người | Reply lịch sự, chỉ dùng từ mượn thông dụng ("refund" thay vì "hoàn tiền"). Người coi là đạt chuẩn (PASS), Judge bắt bẻ thái quá nên chấm FAIL (FP). |
| 6 | H-070 | PASS | FAIL | Người | Dùng emoji cười cợt ("😂😂") trong ngữ cảnh dịch vụ khách hàng là hoàn toàn không phù hợp (FAIL). Judge bỏ sót emoji (FN). |

## Kết luận
- **Tiêu chí bị loại**: `helpful` bị loại thẳng thừng vì độ tin cậy quá thấp:
  - Cohen's Kappa chỉ đạt **0.375** (dưới sàn 0.6), Recall cực kỳ thấp chỉ **0.300** (30% so với sàn 70%).
  - Ma trận nhầm lẫn: `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`. Judge bỏ sót tới 21 lỗi trong tổng số 30 lỗi do người phát hiện (tỷ lệ bỏ lọt lỗi nghiêm trọng lên tới 70%). Một judge bỏ qua hầu hết các vi phạm helpfulness không thể làm gatekeeper trong CI.
- **Tiêu chí được chấp nhận**: `tone_ok` đạt toàn bộ các ngưỡng kiểm chuẩn (κ=0.757, P=0.857, R=0.800). Tỷ lệ lỗi quan sát 28% sau khi hiệu chỉnh Rogan-Gladen theo độ nhạy (80%) và độ đặc hiệu (94.3%) phản ánh tỷ lệ lỗi thực tế xấp xỉ 30.0%.
- **Rủi ro còn lại**: judge chỉ được kiểm trên dữ liệu replay đã công khai; cần nhãn người mới định kỳ để có held-out thật tránh hiện tượng overfit prompt của judge.
