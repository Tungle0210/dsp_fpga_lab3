# Workflow & GitHub guide

## Thành viên bắt đầu
1. Đọc README → SPECIFICATION → TASKS → VERIFICATION.
2. Clone repo, tự tạo feature branch theo nhiệm vụ, tuyệt đối không làm trực tiếp trên main.
3. Mỗi commit rõ: feat/filter, fix/overflow, test/tone, docs/report.
4. Chạy self-checking testbench; lưu log/ảnh/FFT/evidence đúng thư mục.
5. Push branch, mở Pull Request, gắn Issue tương ứng, đợi Leader review.

### Lệnh ví dụ
```bash
git clone https://github.com/Tungle0210/dsp_fpga_lab3.git
cd dsp_fpga_lab3
git checkout -b feat/tv2-treble-control
git add .
git commit -m "feat(treble): add high-shelf biquad"
git push -u origin feat/tv2-treble-control
```

## Pull Request cần có
- Mục tiêu, module sửa, chuẩn I/O và tài liệu tham chiếu.
- Các test đã chạy, simulator, số PASS/FAIL.
- Ảnh waveform/FFT, thông số đo, latency/throughput, Q-format.
- Thay đổi của spec/coefficient cần có lý do và Leader duyệt.
- TV1, TV2 không sửa trực tiếp file thuộc nhau, trừ khi PR giải thích.

## Leader review
- Không được merge khi compile lỗi, chưa có TB, sai interface hoặc chưa xử lý lỗi overflow.
- Phân biệt test trên simulator và hardware.
- Chỉ gắn PASS khi log và số đo thực chứng minh.
- Cập nhật issue sau khi merge.

## Đề xuất release
- **v0.1**: spec + 3 module độc lập + TB
- **v0.2**: audio loopback ADC/DAC PASS
- **v0.3**: integrated EQ + timequest PASS
- **v1.0**: demo + report + ZIP submission

## Quy tắc file
- Commit .qpf/.qsf/.sdc, .sv/.v, scripts, coefficient LUT, ảnh evidence vừa đủ.
- Không commit work/, db/, incremental_db/, simulation cache, .sof quá lớn nếu vượt hạn mức; lưu theo hướng dẫn riêng.
- Không đăng file nhạc có bản quyền, mật khẩu, token, tài liệu nội bộ chưa được phép công bố.
