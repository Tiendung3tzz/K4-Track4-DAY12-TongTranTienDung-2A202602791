# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Tống Trần Tiến Dũng |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602791` |
| GitHub | Tiendung3tzz |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `<điền>` |
| Số seed huấn luyện bạn dùng (`--seeds`) | 3 |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | | | |
| IDM | | | (baseline) |
| EgoMLP | | | |
| PercMLP | | | |
| PercMLP+ (sau khi sửa) | | | |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

Dữ liệu gốc có 992.679 dòng, 9.993 quỹ đạo, lấy mẫu mỗi 0,1 s; độ dài 70–109 bước, trung vị 106 bước (`evidence/data_audit.json`).
Tương quan giữa `Acc_LV` và đạo hàm `Speed_LV` chỉ 0,04535, trong khi cột gia tốc xe theo sau đạt 0,92909; vì vậy `Acc_LV` không đáng tin để tìm sự kiện phanh.
Tôi dùng gia tốc suy ra từ `Speed_LV` đã làm mượt, tránh khuếch đại nhiễu khi lấy đạo hàm và tránh xem một mẫu nhiễu là phanh kéo dài.
Chương trình loại 383 quỹ đạo trùng hệt, còn 9.610; các bản trùng mang ID khác nhau nên nếu không loại trước khi chia, cùng một tình huống có thể lọt vào cả train và test.
Tôi chia theo quỹ đạo vì các khung hình/cửa sổ gần nhau dùng chung lịch sử và tương lai; chia ngẫu nhiên theo khung gây rò rỉ và làm điểm kiểm thử lạc quan giả.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

(trả lời ở đây)

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

(trả lời ở đây)

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

(trả lời ở đây)

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

(trả lời ở đây)

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

(trả lời ở đây)

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

(trả lời ở đây)

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

(trả lời ở đây)
