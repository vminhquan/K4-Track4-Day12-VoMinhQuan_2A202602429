# REPORT — Lab Day 12

> Các tỷ lệ dưới đây được tính từ `evidence/*.json` của lần chạy đầy đủ với MSSV ghi bên dưới. Với mô hình học, dải va chạm là dải qua các seed; cổng dùng seed xấu nhất theo đúng quy tắc.

| | |
|---|---|
| Họ và tên | Võ Minh Quân |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602429` |
| GitHub | `vminhquan` |
| Seed chia dữ liệu / seed mô hình | `6044` / `7496–7500` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `5` |

## Kết quả cuối

| Bộ điều khiển | L2 vòng hở trung bình (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---:|---:|---|
| constant-velocity (`const-vel`) | 1,267 | 20,22 | BLOCK |
| IDM | 1,536 | 0,035 | Baseline |
| EgoMLP | 0,777 | 11,59–25,29 | BLOCK |
| PercMLP | 0,635 | 0,59–4,65 | BLOCK |
| PercMLP+ (sau khi sửa) | 0,818 | 0,17–0,28 | RELEASE theo cổng mô phỏng |

## Câu 1 · Dữ liệu

- `data_audit.json` ghi 992.679 dòng, 9.993 quỹ đạo gốc; loại 383 quỹ đạo trùng hệt, còn 9.610. Trùng ở các ID cách xa nhau có thể rò rỉ cùng một hành vi sang cả tập train lẫn test.
- Tương quan của `Acc_LV` với đạo hàm `Speed_LV` chỉ 0,045, trong khi cột gia tốc xe theo sau đạt 0,929. Vì vậy tôi không dùng `Acc_LV` để xác định ca phanh.
- Tôi dùng `a_lead` suy ra từ `Speed_LV` đã làm mượt; đây là tín hiệu mà `mine_braking` đọc. Chia theo **quỹ đạo** để các khung gần nhau của cùng một lần lái không xuất hiện ở cả hai phía của phép đánh giá.
- Tập sạch có độ dài 70–109 khung mỗi quỹ đạo. Chia theo khung sẽ khiến các cửa sổ chồng lấn và làm điểm kiểm tra lạc quan giả.

## Câu 2 · Phát lại

- Phát lại cùng mã trên 600 đoạn cho 0 đoạn khác bit. Refactor float32 khác bit ở cả 600 đoạn nhưng **0 đoạn vượt dung sai** `atol=rtol=0,001`; độ lệch tuyệt đối lớn nhất là `9,73e-5` m/s². Sai số làm tròn này không phải đổi hành vi.
- AEB đổi lệnh ở 7/600 đoạn. Mở đoạn 72 trong `differing_segments`: các khung 73–79 có TTC dưới ngưỡng AEB 1,5 s và lệnh IDM chưa phanh tới −6 m/s²; chỉ tại đó luật AEB ép lệnh xuống −6. Các khung khác giữ nguyên vì điều kiện kích hoạt không đồng thời thỏa.
- Ở chế độ nhanh nhất, 59.830 khung cho 0 lệnh trễ và 0 khung rớt. Phát theo thời gian thật cho 582 lệnh trễ, 576 khung rớt và 377/600 đoạn đổi lệnh, do thời gian xử lý có lúc vượt hạn của khung kế tiếp.
- Vì chế độ nhanh nhất bỏ qua hạn thời gian, kết quả số học tái lập được ở chế độ đó chưa đủ để kết luận bộ điều khiển đáp ứng đúng hạn khi vận hành.

## Câu 3 · Kịch bản

- Với tốc độ cruise 12 m/s, lưới có 60 khoảng cách (2,76–59,31 m) × 45 tốc độ tiến gần (0–12 m/s), tổng 2.700 ô. Biên vật lý là `gap < dv × t_react + dv²/(2 × 9)`; bộ phanh hết cỡ khớp biên ở cả hai độ trễ, **0 ô lệch** ngoài vùng sát biên.
- Vùng bất khả thi tăng từ 2,22% lên 7,44% khi trễ tăng 0 → 0,6 s, tức tăng 5,22 điểm phần trăm. Diện tích đâm của IDM đúng bằng vùng đó ở mỗi độ trễ; phần đâm còn tránh được của IDM bằng 0% trên lưới này.
- Có 614 quỹ đạo phanh được khai thác (6,39% tập sạch). Mức phanh mạnh ở phân vị 99,9% của người theo sau chỉ 3,625 m/s², trong khi 8,56% ô lưới cần phanh mạnh hơn. Trên `scenarios_idm.png`, các chấm log tập trung ở tốc độ tiến gần thấp; vùng tiến gần cao hầu như trống.
- Do đó phải tách ô **ai cũng không tránh được** khỏi ô còn tránh được nhưng bộ điều khiển vẫn đâm. Tỷ lệ đâm gộp, nếu không trừ biên vật lý, sẽ quy trách nhiệm sai.

## Câu 4 · Vòng hở và vòng kín

| Bộ điều khiển | L2 (m), hạng | Va chạm log trung bình (%), hạng |
|---|---:|---:|
| PercMLP | 0,635 · 1 | 2,24 · 2 |
| EgoMLP | 0,777 · 2 | 20,69 · 4 |
| const-vel | 1,267 · 3 | 20,22 · 3 |
| IDM | 1,536 · 4 | 0,035 · 1 |

- Các cặp đảo thứ tự là const-vel/IDM, const-vel/EgoMLP, IDM/EgoMLP và IDM/PercMLP, đúng như `closedloop.json`. L2 đo khả năng bắt chước quỹ đạo người lái trên trạng thái có sẵn; IDM có mục tiêu giữ khoảng cách nên ít giống người nhưng phanh an toàn khi tự lái.
- EgoMLP giảm L2 so với const-vel nhưng chỉ nhìn lịch sử tốc độ của chính xe; nó thiếu khoảng cách và tốc độ xe dẫn. Khi xe dẫn phanh hoặc cắt vào, cùng lịch sử ego có thể đòi hai hành động khác nhau.
- Lát cắt xe dẫn sắp phanh chiếm 4,93% cửa sổ: L2 của EgoMLP tăng từ 0,777 lên 1,092 m, của PercMLP từ 0,635 lên 0,914 m. Điểm trung bình che mất tình huống quan trọng; vòng kín còn tích lũy sai số qua từng lệnh.

## Câu 5 · Một lần chạy chưa đủ

- Trên 2.883 quỹ đạo test, EgoMLP dao động từ 11,59% đến 25,29% qua 5 seed, rộng 13,70 điểm phần trăm. Nếu chỉ công bố seed 7497 sẽ báo 11,59%; seed 7496 lại cho 25,29%.
- Khoảng Wilson 95% của seed tốt nhất là 10,47–12,80%, của seed xấu nhất là 23,73–26,90%; hai khoảng không chồng nhau. Khác biệt qua seed lớn hơn sai số lấy mẫu của từng lần chạy.
- PercMLP cũng dao động 0,59–4,65%, nên chọn một seed đẹp có thể che rủi ro. Cổng R2 dùng cận trên 95% của seed xấu nhất: PercMLP là 5,48%, vượt tỷ lệ IDM 0,035% cộng biên 1 điểm.
- Vì vậy tôi báo dải qua seed và khoảng tin cậy, đồng thời đọc kết quả lát cắt và lưới tình huống trước khi kết luận.

## Câu 6 · Tác nhân phản ứng

Ở độ trễ 0,6 s, bảng dưới là tỷ lệ ô có **bất kỳ va chạm nào**; số trong ngoặc là va chạm từ phía sau (`agents.json`).

| Bộ điều khiển | Xe sau phát lại (%) | Xe sau phản ứng (%) |
|---|---:|---:|
| const-vel | 77,63 (0,00) | 77,63 (0,00) |
| IDM | 75,74 (68,30) | 7,44 (0,00) |
| EgoMLP | 80,48 (0,00) | 80,48 (0,00) |
| PercMLP | 76,78 (42,15) | 34,63 (0,00) |
| PercMLP+ | 75,30 (67,37) | 7,93 (0,00) |

- Xe sau IDM được hiệu chỉnh `T=1,4 s`: khoảng cách Wasserstein-1 tới phân phối người lái là 0,339, tốt hơn xe không phản ứng 0,858; trung vị headway người lái là 1,859 s. Đây là khớp phân phối khoảng cách, chưa phải xác thực đầy đủ hành vi xe sau.
- Trong thế giới phát lại, xe sau cứ giữ vận tốc cũ dù IDM phanh để né xe trước; 68,30% ô bị tính đâm phía sau. Khi xe sau phản ứng, con số ấy về 0,00% và tỷ lệ va chạm bất kỳ của IDM từ 75,74% xuống 7,44%.
- Const-vel hầu như không gây va chạm sau nhưng vẫn đâm trước ở 77,63% ô. Chỉ dùng xe sau phát lại sẽ làm IDM trông gần như const-vel; mô hình tác nhân có phản ứng làm đổi hẳn kết luận.

## Câu 7 · Cổng phát hành và sửa

- PercMLP bị chặn bởi R2–R6: cận trên Wilson log 5,48% so với IDM 0,035% + 1 điểm; trên ca phanh là 15,66% (cận trên 21,37%) so với IDM 0,51% + 5 điểm. Tỷ lệ qua seed 0,59–4,65% còn trượt R5.
- Nguyên nhân thiếu phủ kịch bản thể hiện ở R4: 27,19% ô cắt đầu còn tránh được vẫn đâm so với IDM 0%, và R6: 34,63% va chạm trong thế giới xe sau phản ứng so với IDM 7,44%. Nguyên nhân khác là sai số bắt chước nhỏ tích lũy khi tự lái, thể hiện ở R2, R3, R5.
- Tôi bổ sung 20.000 kịch bản cắt đầu với nhãn IDM và chạy 2 vòng DAgger trên trạng thái do chính mô hình đi vào. PercMLP+ giảm vùng đâm còn tránh được xuống 0,48%, va chạm log còn 0,17–0,28%; R2–R6 đều chuyển sang qua và cổng ra `RELEASE`.
- L2 vòng hở tăng 0,635 → 0,818 m, nhắc rằng bắt chước log tốt hơn không đồng nghĩa điều khiển an toàn hơn. PercMLP+ học nhãn từ IDM và chỉ chứng minh **không kém hơn theo các ngưỡng đã chọn**, không chứng minh tốt hơn IDM.
- Ở tốc độ giữ lại 20 m/s, vùng đâm còn tránh được của PercMLP+ vẫn dao động 0–2,37% trong khi IDM là 0%. Tôi chưa đề nghị thay IDM ngay; cần thử nghiệm độc lập ngoài tập dữ liệu, kiểm thử cảm biến và phần cứng, rồi mới xét phát hành thực tế.

## Câu 8 · Bài lab này không chứng minh được gì

- Mô phỏng chỉ xét chuyển động dọc một chiều; không có đánh lái, giao lộ hay người đi bộ. Lưới 2.700 ô cắt đầu không đại diện cho toàn bộ xung đột giao thông, nên kết quả `RELEASE` chưa chứng minh an toàn ngoài đường.
- Cảm nhận trong mô phỏng gần như lý tưởng; phép thử nhiễu chỉ thêm sai số khoảng cách 1 m và tốc độ xe dẫn 0,5 m/s. Trễ cảm biến, mất quan sát và lỗi nhận dạng có thể làm điều kiện TTC hoặc lệnh phanh sai đúng lúc nguy hiểm nhất; cần thử hệ cảm biến thật và tiêm lỗi có hệ thống.
- Xe dẫn trên log bị phát lại, không phản ứng với xe thử. Xe sau phản ứng chỉ là IDM hiệu chỉnh `T=1,4 s`, dù W1 giảm còn 0,339; tương tác nhiều xe và hành vi tài xế khác có thể đổi thứ hạng. Cần mô phỏng đa tác nhân đã kiểm chứng và thử nghiệm có giám sát.
- Tập huấn luyện kịch bản chỉ thấy cruise tới 16 m/s; ở 20 m/s PercMLP+ còn 0–2,37% vùng đâm có thể tránh, so với IDM 0%. Cần kiểm thử ngoài miền, các điều kiện đường và thời tiết khác, nhiều seed và khoảng tin cậy trước quyết định triển khai.
