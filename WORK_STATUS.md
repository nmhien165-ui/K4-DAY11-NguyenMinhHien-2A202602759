# WORK STATUS

## Mục tiêu hiện tại
Hoàn thiện Lab 11 và bằng chứng nộp bài; người dùng tự đóng gói folder cuối.

## Việc đã hoàn thành
- Hoàn thành 10 TODO trong `lab11/`; chạy `python -m pytest -q tests/test_todos.py`: 31/31 passed.
- Chạy pipeline A–F đầy đủ, không `--quick`, MSSV `202602759`, town C; chạy G đủ 1.500 lượt và F export sau mine.
- `evidence/` có meta, world, replay, mc, hunt, cloud, estimate, fix, mine, viewer; các bước chính đều báo `_run.quick=false`.
- Mine giữ 1.491/1.500 mẫu (`accepted=0.994`); baseline có 3 avoidable; policy chọn không có hit trong mẫu. Cảnh dựng chính xác ghi ego–C2 lúc 12,65 s.
- Hoàn thiện 9 câu trong `REPORT.md`, đối chiếu số với JSON; không khai khống phần tự lái bằng tay.
- Lưu 4 ảnh viewer thật trong `evidence/shots/`: bus_bypass, crosswalk_block, replay_rear, hazard_brake.
- `git diff --check` không báo lỗi whitespace.

## Việc đang làm
- Hoàn tất đối chiếu file cuối và báo người dùng các phần cần tự làm.

## File đã sửa/tạo
- `lab11/metrics.py`, `lab11/feasible.py`, `lab11/estimate.py`, `lab11/search.py`, `lab11/modes.py`, `lab11/policy.py`.
- `hazard.json`, `REPORT.md`, `WORK_STATUS.md`, `evidence/shots/*.png`.
- Pipeline tạo `evidence/*.json`.

## Quyết định quan trọng
- Không sửa `avsim/`, `run_all.py`, `tests/`, phân phối tự nhiên, seed.
- Chạy toàn bộ cục bộ; không dùng Colab/Kaggle.
- Exposure 0,0023/trip là proxy rộng NHTSA, không phải số đo trực tiếp cho cut-in kèm phanh; report ghi rõ giới hạn.
- Không tạo folder `K4-DAY11-HoVaTen-MSSV` vì người dùng tự đóng gói.

## Lỗi/vấn đề còn tồn tại
- Bài chưa đóng gói theo tên folder cuối và chưa nộp VLearn; đây là phần người dùng sẽ làm.
- Không thực hiện thử nghiệm người dùng tự lái; REPORT nêu rõ không có dữ liệu.

## Bước tiếp theo cần làm
Người dùng tự đóng gói các file cần thiết theo mẫu khóa học, kiểm tra tên MSSV/họ tên, rồi nộp trên VLearn.
