MSSV: 202602759

# Lab 11 · Scenario Forge — báo cáo

Các số dưới đây được lấy từ lần chạy đầy đủ (`_run.quick=false`) ở `evidence/`, thị trấn C. Ảnh viewer nằm trong `evidence/shots/`.

## 1. Phát lại log có trung thực không?

`replay.json`: lấy 400 kịch bản; bỏ 0 log có va chạm của người lái thật, nên dùng 400. Chạy trên log phát lại có 132 va chạm (115 `rear`, 17 `side`); chạy trong thế giới phản ứng có 0, và `crash_in_both=0`. Phần lớn va chạm sau là giả vì xe phía sau trong log cứ đi theo quỹ đạo đã ghi, không phản ứng với việc planner phanh hoặc nhường sớm hơn người lái thật. Trong kịch bản 0, `replay_only[0]` là va chạm `rear`, trong khi TTC nhỏ nhất của lần chạy phản ứng là 2,217 s: xe theo log không tự tạo khoảng cách mới để né cú giảm tốc của ego.

## 2. Kịch bản bất khả thi

`lab11/feasible.py:is_feasible` loại xe chuyển làn nếu gia tốc ngang cần cho chuyển làn cosine 3,5 m vượt `mu·g` (độ bám lốp); loại nếu phanh sau khi nhập làn vượt `mu·g`; nếu người đi bộ chạy nhanh hơn 2,5 m/s; hoặc xe nhập làn có `cutV` ngoài khoảng `(0, 22]` m/s. `cutDur≤0` cũng không thể dùng trong công thức nên bị loại. Trong `hunt.json`, `rear_ended` có nhiều mẫu không khả thi nhất: 4, so với 3 ở `bus_bypass`, 2 ở `crosswalk_block`, 0 ở `cut_in`. CEM tối ưu điểm nguy hiểm trong vùng đề xuất nên các tổ hợp cực đoan dễ được đẩy lên trước bộ lọc vật lý. Nếu đếm chúng như va chạm thật, xác suất bị thổi phồng bởi những tình huống không thể xảy ra; pipeline đã lọc chúng và không tính vào số lỗi.

## 3. Monte Carlo cho bạn biết gì?

`mc.json`: `k=9`, `n=3000`, `p̂=0,003`, khoảng Wilson 95% `[0,001579; 0,005692]`; cả 9 va chạm được gán `bus_bypass`. Với xấp xỉ nhị thức iid, để sai số nửa khoảng bằng 20% của p cần khoảng `1,96²(1−p)/(0,2²p) ≈ 31.918` lần chạy, tức khoảng 10,6 lần ngân sách MC hiện tại. `cut_in`, `crosswalk_block`, `rear_ended` và `other` đều có 0 hit. Với MC 3.000, cận Wilson trên của mỗi kiểu zero-hit trong `mc.json:per_mode.*.hi` là 0,001279, không phải kết luận xác suất bằng 0. Riêng ước lượng importance sampling dùng quy tắc zero-hit bảo thủ `3/(0,2·n)`; với `n=600` là 0,025 cho mỗi kiểu/seed.

## 4. Importance sampling, ESS và độ suy biến

`estimate.json` dùng `n=600` cho mỗi kiểu và seed; dưới đây p, hits, ESS và `top_share` (làm tròn):

| Kiểu | Seed | p | Hits | ESS | Top share |
|---|---:|---:|---:|---:|---:|
| bus_bypass | 0 | 4,623e−3 | 26 | 120,9 | 0,660 |
| cut_in | 0 | 0 | 0 | 122,6 | 0 |
| crosswalk_block | 0 | 5,914e−7 | 102 | 124,1 | 0,242 |
| rear_ended | 0 | 0 | 0 | 129,3 | 0 |
| bus_bypass | 1 | 8,344e−4 | 24 | 110,0 | 0,362 |
| cut_in | 1 | 0 | 0 | 115,2 | 0 |
| crosswalk_block | 1 | 1,142e−5 | 102 | 108,1 | 0,774 |
| rear_ended | 1 | 0 | 0 | 128,4 | 0 |
| bus_bypass | 2 | 1,260e−3 | 27 | 130,1 | 0,257 |
| cut_in | 2 | 0 | 0 | 130,0 | 0 |
| crosswalk_block | 2 | 2,383e−7 | 87 | 105,1 | 0,160 |
| rear_ended | 2 | 0 | 0 | 109,9 | 0 |

Ba seed không đồng ý tốt: tổng theo seed lần lượt là 0,004624; 0,000846; 0,001260. MC cho 0,003; seed 0 cùng bậc, hai seed còn lại thấp hơn vài lần. `top_share>0,5` ở `bus_bypass` seed 0 (0,660) và `crosswalk_block` seed 1 (0,774): một mẫu mang phần lớn ước lượng, vì vậy số đó nhạy với đúng một trọng số, dù ESS tổng không quá thấp. `estimate.json:budget=16.800`, thấp hơn khoảng 31.918 lần MC cần cho ±20%, nhưng IS ở đây chưa chứng minh độ chính xác tương đương. Phần `other` lấy từ `mc.json` (`k=0`, `n=3000`, cận trên 0,001279); không săn `other` vì CEM cần một nhãn thất bại và score riêng, còn `other` gom các va chạm không thuộc các nhãn mục tiêu.

## 5. Các kiểu thất bại

- `bus_bypass`: [ảnh B-01](evidence/shots/bus_bypass.png). Xe buýt che góc nhìn, `pedLook=0`, mặt đường ướt (`μ=0,5`); xe ta đi đến vùng khuất trong khi xe nhập làn chọn khe 6,1 m. Viewer ghi va chạm với xe cắt làn lúc 12,7 s.
- `crosswalk_block`: [ảnh C-01](evidence/shots/crosswalk_block.png). Xe ta dừng thân xe trên vạch; người đi bộ không bước vào xe và phải đợi, nên đây là lỗi chặn lối chứ không phải va chạm hộp xe. Cảnh viewer đánh dấu lúc 22,0 s.
- `replay` (chỉ va chạm khi log phát lại): [ảnh R-01](evidence/shots/replay_rear.png). Xe bám đuôi giữ quỹ đạo log và đâm sau lúc 20,3 s; trong thế giới phản ứng, xe đó có thể điều chỉnh theo ego.

Cuộc săn `cut_in` không tìm được va chạm đúng nhãn: nó chỉ lưu 35 `bus_bypass`; cuộc săn `rear_ended` chỉ có 102 `bus_bypass` và 1 `crosswalk_block`, không có `rear_ended`. Đây chỉ nói về ngân sách và seed này, không chứng minh các kiểu đó không tồn tại. Nếu gộp mọi nhãn vào một cuộc săn, CEM có thể tập trung vào loại dễ ghi điểm nhất (`bus_bypass`) và bỏ sót `crosswalk_block`; các `found` ở bốn cuộc săn cho thấy cần mục tiêu riêng.

## 6. Sửa planner

`fix.json:chosen.policy` chọn `cutinLat=2.6`. Trên TRAIN: `fail=1,488e−6`, `fail_hi=3,084e−6`, `progress=211,936 m`, `harsh=8,391`. Trên TEST: bản sửa có `fail=2,069e−5`, `fail_hi_seen=4,693e−5`, `fail_hi=0,001326`, `progress=210,941 m`, `harsh=8,667`; policy gốc có `fail=0,0006388`, `fail_hi_seen=0,001040`, `fail_hi=0,002319`, `progress=211,192 m`, `harsh=8,684`. Bản sửa giảm quãng đường trung bình khoảng 0,251 m; số khung phanh gấp gần như ngang (thấp hơn 0,018). `fail_hi` vẫn có thể lớn hơn `fail` của baseline vì cộng cận trên cho các kiểu zero-hit/`other`; đó là bất định còn lại chứ không phải số va chạm đã quan sát.

[Ảnh B-01 ở đúng va chạm](evidence/shots/bus_bypass.png) cho thấy lần chạy policy gốc va chạm lúc 12,7 s còn quỹ đạo so sánh của policy được chọn không va chạm. Ứng viên `cautionV=5` nghe an toàn vì đặt tốc độ thấp hơn gần vùng khuất, nhưng `fix.json:table` cho `fail=0,008567`, `fail_hi=0,025357`, `progress=171,126 m`; mẫu bus-bypass duy nhất còn `top_share=1`. Đây là tín hiệu đáng xem chứ chưa phải kết luận nhân quả chắc chắn. Trong thế giới phản ứng, ego giảm tốc/đổi nhịp sẽ làm IDM và thời điểm nhập làn của xe khác đổi theo; giảm một tốc độ không chỉ thay quỹ đạo ego. Cần thêm mẫu độc lập để xác nhận cơ chế.

## 7. Từ mô phỏng ra đường thật

1. Thị trấn C giả định trung vị khe nhập làn `cutGap=10 m` (`space.py`). Thay bằng phân bố khe đo từ camera/radar hoặc dữ liệu quỹ đạo xe tự nhiên, phân tầng theo tốc độ và loại đường. Nếu khe thật nhỏ hơn, mô phỏng có thể đánh giá thấp xung đột nhập làn; nếu lớn hơn, có thể đánh giá cao. Dữ liệu 100-Car NHTSA có phân loại sự kiện đổi làn nhưng không phải mẫu đại diện cho đường Việt Nam.
2. Xác suất người đi bộ ngó qua xe buýt `pedLook=0,80` là giả định của town C. Thay bằng video tại điểm dừng xe buýt/giao cắt, mã hoá việc nhìn, vị trí che khuất và thời điểm bước xuống. Tỉ lệ nhìn thật thấp hơn 0,80 có thể làm tăng các lần băng qua muộn và lỗi chặn lối; tỉ lệ cao hơn có thể làm giảm chúng.
3. Town C giả định đường ướt chiếm 20%, với `μ=0,5` (đường khô `μ=0,8`). Thay bằng nhật ký mưa kết hợp đo độ bám tại các tuyến và telemetry phanh của đội xe. Nếu đường ướt thường xuyên hơn hoặc độ bám thấp hơn, quãng dừng dài hơn và nguy cơ trong mưa hiện bị đánh giá thấp; nếu điều kiện thực tế khô/độ bám cao hơn, nguy cơ có thể bị đánh giá cao.

Hazard riêng dùng NHTSA 100-Car Phase II làm proxy rộng: 64 near-crashes + 324 incidents có cut-in trên khoảng 2.025.000 miles, quy đổi với 12 miles/trip. Đây không phải tỉ lệ trực tiếp của tình huống kết hợp cut-in rồi phanh; cần dữ liệu đội xe địa phương gắn nhãn chính xác tình huống và odometer để thay giả định exposure. Nguồn: [NHTSA DOT HS 810 593](https://www.nhtsa.gov/sites/nhtsa.gov/files/100carmain.pdf) và [NHTSA, Modeling Driver Response to Lead Vehicle Decelerating](https://www.nhtsa.gov/document/modeling-driver-response-lead-vehicle-decelerating).

## 8. Giới hạn

Các xác suất này chỉ nói về một phố mô phỏng và một phân phối dạy học cho town C; chúng không phải tần suất tai nạn ngoài đường. Xe khác chỉ phản ứng theo IDM và người đi bộ theo luật đơn giản, nên không đại diện đầy đủ hành vi người thật. Nhận biết được xem như hoàn hảo ngoài vùng bị xe buýt che; không mô hình hoá lỗi cảm biến, thời tiết/ánh sáng rộng hơn, hay khác biệt giữa xe.

Nếu có thêm một tuần, tôi sẽ thu trajectory đã đồng bộ từ camera/radar và telemetry trên vài tuyến, mã hoá cut-in, phanh sau nhập làn, tầm nhìn người đi bộ và thời tiết/độ bám; sau đó fit lại phân phối và chạy lại cùng pipeline trên tuyến/tuần chưa dùng để kiểm tra khả năng tổng quát hoá.

## 9. Rắc rối của bạn

- **Rắc rối:** xe C2 nhập làn trước ego rồi phanh (`hazard.json:hazards[0]`: 1,2 m/s² trong 2,2 s; trigger 25 m). Đây là xung đột cut-in có giảm tốc ngay sau khi nhập.
- **Căn cứ và giả định:** xe cắt làn gần đầu xe rồi phanh tạo thời gian phản ứng ngắn. `hazard.json:space` quét `decel=[0,8;1,5]`, `dur=[1,5;3]`, `t0=[0;12]`, `trig=[15;50]`; giới hạn thời gian/trigger là dải dựng kịch bản trong `avsim/studio.py`, không phải phân phối đo được. `exposure=0,0023/trip` là proxy rộng NHTSA đã giải thích ở câu 7 và `hazard.json:why`.
- **Kết quả:** `mine.json` giữ 1.491/1.500 mẫu khả thi (`accepted=0,994`; 9 mẫu bị loại). Policy gốc có `avoidable=3`, `unavoidable=0`, `unattributed=0`, `low_speed=0`; `P(lỗi|hazard)=0,002012` với Wilson 95% `[0,000685;0,005899]`; tần suất quy đổi `4,628e−6/trip`, cận trên `1,357e−5`. Policy được chọn có 0 va chạm ở 1.491 mẫu; `P̂=0`, Wilson `[0;0,002570]`; tần suất điểm 0, cận trên `5,911e−6/trip`. Mẫu dựng chính xác va chạm ego–C2 lúc 12,65 s (`mine.json:exact.planner`). Trong MC tự nhiên, va chạm duy nhất được thấy là `bus_bypass`, `9/3000=0,003` (`mc.json`); số hazard quy đổi nhỏ hơn khoảng 648 lần, nhưng đây là so sánh giữa một proxy exposure hẹp và xác suất mô phỏng tự nhiên, không phải hai ước lượng cùng phạm vi ngoài đời.
- **Hình:** [hazard_brake.png](evidence/shots/hazard_brake.png), chương 9, tại khung va chạm 12,7 s; xe chạy policy gốc va chạm, policy được chọn trong bóng so sánh không va chạm.
- **Tự lái:** tôi không thực hiện thử nghiệm cầm lái trong viewer, nên không khẳng định mình sẽ tránh được.
- **Cách sửa:** trong sweep này, policy `cutinLat=2.6` có 0 lỗi quan sát được so với 3 của baseline; tăng ngưỡng ngang có thể nhận xe đang nhập làn như xe dẫn trước sớm hơn. Tuy nhiên cận zero-hit vẫn còn và đây chưa phải bảo đảm. Planner nên dự đoán đồng thời quỹ đạo nhập làn và giảm tốc của xe đó, rồi kiểm tra khoảng dừng theo độ bám và tạo khoảng cách trước khi C2 cắt vào; nếu không đủ, giảm tốc từ sớm và giữ đường thoát thay vì chỉ phản ứng khi đã thành xe dẫn.
