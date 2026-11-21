# TASK ASSIGNMENTS – LAB 3

**Công nghệ bắt buộc: Verilog/SystemVerilog cho cả RTL và testbench, không Python/MATLAB.** Xem [kiểm thử thuần SystemVerilog](RTL_ONLY_VERIFICATION.md).

**Nguyên tắc độc lập:** TV1 và TV2 chỉ dùng [SPECIFICATION.md](SPECIFICATION.md) và [COEFFICIENTS_Q5_27.md](COEFFICIENTS_Q5_27.md); không dùng module/RTL của nhau trong quá trình phát triển. TV3 ghép ba module ở top. Leader kiểm tra chất lượng và tích hợp.

## TV1 – Bass + Mid IIR (không phụ thuộc TV2)

**Đường dẫn:** rtl/bass_mid/, tb/, doc/simulation/.

### Công việc
1. Viết bass_eq.sv: Low-Shelf 250Hz, S=1, gain_sel 0..8.
2. Viết mid_eq.sv: Peaking EQ 1kHz, Q=0.75, gain_sel 0..8.
3. Tạo tb_bass_eq.sv, tb_mid_eq.sv, không lệ thuộc code TV2/TV3.
4. Kiểm tra signed arithmetic Q4.28/Q5.27, rounding/saturation, reset/state.
5. Xác minh gain 0/±3/±6/±9/±12, impulse, tone/multi-tone/edge values.
6. Dùng **SystemVerilog testbench** tạo single-tone/multi-tone, đo đáp ứng biên độ bằng sine/cosine correlation; so với đáp ứng kỳ vọng của bảng hệ số. Ghi rõ latency và cách thay coefficients an toàn. Không dùng Python.

### Bàn giao
- RTL + SystemVerilog testbench; log PASS/FAIL; waveform có chú thích; log/CSV xuất từ testbench nếu cần; mô tả nguyên lý + fixed-point + latency.
- PR lên nhánh feat/tv1-bass-mid.

**PASS khi:** Hai module có testbench tự kiểm tra, không mất/đảo sample; đáp ứng phù hợp reference trong giới hạn [VERIFICATION.md](VERIFICATION.md).

## TV2 – Treble IIR + Control (không phụ thuộc TV1)

**Đường dẫn:** rtl/treble_control/, tb/, doc/simulation/.

### Công việc
1. Viết treble_eq.sv High-Shelf 4kHz S=1 và tb_treble_eq.sv.
2. Viết gain_controller.sv, tb_gain_controller.sv: 3 gain_sel, chọn band bằng SW[1:0], KEY[1] tăng, KEY[2] giảm, KEY[3] bypass, KEY[0] reset.
3. Debounce các KEY; mỗi nhấn tăng/giảm đúng 1 mức, bão hòa tại 0 và 8.
4. Test reset cả 3 gain về 4, selected band độc lập, giữ trạng thái sau khi đổi band.
5. Test Treble 9 mức gain và cross-band 100Hz/1kHz/10kHz.
6. Bàn giao mô tả mapping SW/KEY, latency và cách load hệ số tại sample boundary.

### Bàn giao
- RTL + 2 SystemVerilog testbench + log PASS/FAIL + waveform/kết quả đo gain + bảng control.
- PR lên nhánh feat/tv2-treble-control.

**PASS khi:** Treble đúng reference; control hoạt động độc lập, không sinh nhiều xung mỗi nhấn; không có kết nối tới RTL TV1.

## TV3 – CODEC + Quartus + Integration

**Đường dẫn:** rtl/audio/, rtl/common/, rtl/top/, quartus/, tb/, doc/synthesis/, doc/hardware_demo/.

### Công việc
1. Khảo sát Lab 1: audio_pll, wm8731_config, wm8731_i2s_tx, I2C, synchronizer.
2. Bổ sung I²S RX/ADC, **bật đường analog Line-In/ADC** ở WM8731 config. Kiểm tra master/slave và ADC/DAC frame timing.
3. Trước tiên xây dựng và test ADC→DAC direct loopback trên board.
4. Viết equalizer_top.sv kết nối Bass→Mid→Treble + gain controller + bypass, buffer/CDC chống mất mẫu.
5. Tạo .qpf/.qsf/.sdc, gán pin ADC; TimeQuest và tài nguyên.
6. Demo laptop/phone → DE10 Line-In → Line-Out → loa, thay đổi gain ba band.

### Bàn giao
- RTL + integrated TB, Quartus source project, sof nạp được, ảnh resources/TimeQuest, video/audio demo.
- PR lên nhánh feat/tv3-audio-integration.

**PASS khi:** loopback và 3-band chạy trên kit thật; không drop sample; KEY/SW thay gain real-time; synthesis/timing có số liệu thật.

## LEADER – Project Management + Technical QA

1. Chốt SPEC, theo dõi tiến độ và giữ interface chung.
2. Duyệt PR của TV1/2/3 và chỉ merge khi evidence có kiểm chứng.
3. Review các **testbench SystemVerilog tự kiểm tra**, phép đo gain/đáp ứng tần số và sai số fixed-point; **không yêu cầu Python hoặc MATLAB**.
4. Phân tích frequency response, overflow, transient, CDC, latency/timing/resource.
5. Điều phối buổi nạp DE10 và demo, tổng hợp ảnh/video + soạn báo cáo.
6. Nộp ZIP lên LMS. Chỉ làm bonus sau khi yêu cầu 3-band PASS.

**Tiến độ:** M1 spec → M2 module TB → M3 loopback → M4 integrate → M5 synthesis/demo → M6 report. Hạn chót cụ thể lấy theo LMS.
