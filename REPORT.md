# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Tống Trần Tiến Dũng |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602791` |
| GitHub | Tiendung3tzz |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | Chia dữ liệu: 1043; mô hình: 3173, 3174, 3175 |
| Số seed huấn luyện bạn dùng (`--seeds`) | 3 |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1,26713 | 19,52827 | |
| IDM | 1,50938 | 0,06937 | (baseline) |
| EgoMLP | 0,75714–0,76447 | 12,34825–32,53555 | |
| PercMLP | 0,62005–0,62597 | 0,24280–0,45092 | |
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

Cùng mã chạy lại giống hệt cả 600 đoạn; refactor float32 khác bit ở 600/600 đoạn nhưng 0 đoạn vượt dung sai, sai khác lớn nhất 0,000403132 m/s² (`evidence/replay.json`).
Đổi độ chính xác và thứ tự phép toán gây sai số làm tròn; tôi chấp nhận `|new−golden| ≤ 0,001 + 0,001×|golden|` m/s², còn hai NaN đệm cùng vị trí được coi bằng nhau.
AEB làm lệch 12/600 đoạn; ở đoạn 52 chỉ các khung 98–101 thay đổi, TTC lần lượt 1,4743; 1,4333; 1,4311; 1,4896 s, đều dưới ngưỡng 1,5 s (`evidence/replay_inspection.json`).
Ví dụ khung 98 có gap 4,3033 m, tiến gần 2,9189 m/s: lệnh −5,4363 thành −6 m/s²; AEB chỉ đổi lệnh khi đang tiến gần, TTC thấp và lệnh cũ chưa phanh ít nhất −6 m/s².
Trên 59.774 khung, chế độ nhanh nhất có 0 trễ/0 rớt; phát theo thời gian thật giả lập có 582 khung trễ, 576 khung rớt, làm 368/600 đoạn vượt dung sai.
Chi phí xử lý đột biến vượt chu kỳ 100 ms khiến phải giữ lệnh cũ và bỏ thông điệp đến khi bộ điều khiển còn bận; phát nhanh nhất không áp hạn chót nên che lỗi này. Đây là mô phỏng lịch xử lý, không phải đo độ trễ phần cứng Colab.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Với cruise 12 m/s, lưới gồm 60 khoảng cách từ 2,76005 đến 59,31233 m và 45 tốc độ tiến gần từ 0 đến 12 m/s, tổng 2.700 ô (`evidence/scenarios.json`).
Biên bất khả thi là `gap < dv×t_react + dv²/(2×9)` với `dv > 0`: mất khoảng cách trong thời gian trễ cộng quãng đường cần để triệt tiêu tốc độ tiến gần khi phanh cực đại.
Ở độ trễ 0 s có 2,2222% ô bất khả thi; tại 0,6 s là 7,4444%. Vùng đâm của IDM tăng tương ứng từ 2,2222% lên 7,4444%, tức 5,2222 điểm phần trăm; phần va chạm có thể tránh nhưng IDM gây ra bằng 0 ở cả hai độ trễ.
Full-brake khớp biên vật lý với 0 ô lệch ngoài dải sát biên ở cả hai độ trễ, nên không quy toàn bộ vùng đâm cho lỗi điều khiển.
Khai thác được 614 sự kiện phanh/9.610 quỹ đạo (6,3892%); giới hạn phanh người lái tham chiếu p99,9 là 3,62484 m/s², trong khi cơ cấu chấp hành cho phép 9 m/s²; 8,5556% lưới đòi hỏi mạnh hơn giới hạn tham chiếu này.
Các chấm trắng trong `evidence/scenarios_idm.png` tập trung ở tốc độ tiến gần thấp, phần lớn xa vùng tốc độ tiến gần cao sát biên va chạm; log có sự kiện phanh nhưng phủ thưa các ca cắt vào khó. Giá trị p99,9 là phân vị tham chiếu, không phải cực đại tuyệt đối của mọi mẫu.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

Số dưới đây dùng trung bình qua ba seed để xếp hạng, cùng 2.883 quỹ đạo test; hạng 1 là tốt nhất (`evidence/openloop.json`, `closedloop.json`).

| Bộ điều khiển | L2 trung bình (m) | Hạng vòng hở | Va chạm trung bình (%) | Hạng vòng kín |
|---|---|---|---|---|
| constant-velocity | 1,26713 | 3 | 19,52827 | 3 |
| IDM | 1,50938 | 4 | 0,06937 | 1 |
| EgoMLP | 0,76069 | 2 | 19,94450 | 4 |
| PercMLP | 0,62309 | 1 | 0,36998 | 2 |

Bốn cặp đổi thứ tự là constant-velocity/IDM, constant-velocity/EgoMLP, IDM/EgoMLP và IDM/PercMLP; đây là thứ hạng thực nghiệm của các trung bình, chưa phải kiểm định ý nghĩa thống kê của từng cặp.
IDM giữ khoảng cách và phanh theo tình huống thay vì bắt chước chính xác quỹ đạo người lái; dự báo của luật còn giả sử xe dẫn giữ tốc độ. L2 đo độ giống vị trí người lái nên có thể phạt một cách lái khác nhưng ít va chạm.
EgoMLP chỉ thấy lịch sử tốc độ bản thân, thiếu gap và tốc độ xe dẫn để nhận biết nguy cơ. Khi tự lái, sai số làm lịch sử lệch khỏi dữ liệu huấn luyện rồi tích luỹ; ngoại suy tốt trên log không bảo đảm phản ứng an toàn.
Lát cắt xe dẫn sắp phanh chỉ chiếm 4,1646% cửa sổ test; L2 của EgoMLP tăng từ 0,76069 lên 1,04670 m và PercMLP từ 0,62309 lên 0,82090 m, cho thấy trung bình che các tình huống khó.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

Ba seed EgoMLP 3173, 3174, 3175 lần lượt gây 356, 431, 938 va chạm trên cùng 2.883 quỹ đạo, tương ứng 12,34825%; 14,94971%; 32,53555% (`evidence/closedloop.json`).
Dải tốt nhất–xấu nhất rộng 20,18730 điểm phần trăm, dù L2 chỉ dao động 0,75714–0,76447 m; điểm ngoại tuyến gần nhau không đồng nghĩa chính sách tự lái ổn định.
Wilson 95% của seed tốt nhất là [11,19718%; 13,59953%], còn seed xấu nhất là [30,84956%; 34,26803%]; hai khoảng không chồng nhau, nên khác biệt giữa các seed lớn hơn nhiều mức bất định lấy mẫu thể hiện ở đây.
Nếu chỉ chọn seed 3173, tôi có thể báo 12,34825% và kết luận EgoMLP ít va chạm hơn constant-velocity 19,52827%; chọn seed 3175 lại cho 32,53555% và kết luận ngược.
Vì vậy cần công bố mọi seed, dải kết quả và khoảng tin cậy; cổng dùng seed xấu nhất và kiểm tra độ ổn định, không lựa seed đẹp để phát hành.
Wilson mô tả bất định tỷ lệ trên các quỹ đạo trong điều kiện mô phỏng, không đo mọi thay đổi ngoài đời; ba seed dùng chung tập test nên cũng không thể xem như ba tập quan sát độc lập rồi gộp tuỳ tiện.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

Bảng ghép hai thế giới tại độ trễ xe thử 0,6 s; xe sau cách 18,8 m, trễ phản ứng 0,6 s. Mỗi mô hình học dùng hàng của seed có tỷ lệ va chạm bất kỳ trong thế giới phản ứng cao nhất (`evidence/agents.json`).

| Bộ điều khiển | Xe sau phát lại: bất kỳ (%) | Xe sau phát lại: phía sau (%) | Xe sau phản ứng: bất kỳ (%) | Xe sau phản ứng: phía sau (%) |
|---|---|---|---|---|
| constant-velocity | 77,6296 | 0 | 77,6296 | 0 |
| IDM | 75,7407 | 68,2963 | 7,4444 | 0 |
| EgoMLP | 76,8889 | 0 | 76,8889 | 0 |
| PercMLP | 77,8519 | 56,2222 | 21,6667 | 0 |

Hiệu chỉnh chọn `T = 1,4 s`, có Wasserstein-1 giữa phân bố headway mô phỏng và người lái thấp nhất trong các T thử: 0,36263 s, so với 0,80162 s ở xe không phản ứng; headway trung vị người lái là 1,86442 s. Đây là khớp một phân bố hành vi, chưa chứng minh giống mọi tài xế.
Khi IDM phanh tránh xe cắt vào, xe sau phát lại vẫn giữ tốc độ cũ và đâm vào xe thử: 68,2963% ô bị tính va chạm sau. Con số này về 0 khi xe sau phản ứng, cho thấy giả định phát lại tạo ra va chạm không phù hợp với tác nhân đã hiệu chỉnh, không chứng minh mọi va chạm sau ngoài đời đều tránh được.
Trong bộ số này IDM vẫn xếp trên constant-velocity ở cả hai thế giới, nhưng khoảng cách tỷ lệ va chạm bất kỳ tăng từ 1,8889 lên 70,1852 điểm phần trăm; không có đảo hạng giữa hai bộ này. PercMLP mới là cặp đảo hạng với constant-velocity: từ 77,8519% > 77,6296% thành 21,6667% < 77,6296%.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

(trả lời ở đây)

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

(trả lời ở đây)
