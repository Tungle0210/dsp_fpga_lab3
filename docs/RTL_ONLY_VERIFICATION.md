# Kiểm thử Lab 3 bằng Verilog/SystemVerilog — KHÔNG PYTHON

**Áp dụng cho TV1, TV2, TV3 và Leader.** Mọi RTL tổng hợp và testbench của Lab 3 đều được viết bằng **Verilog/SystemVerilog**. Không dùng Python/MATLAB để tính hoặc chấm kết quả.

## 1. Phân biệt RTL và testbench

| Việc | Dùng ở đâu | Công nghệ |
|---|---|---|
| Biquad Bass/Mid/Treble, ROM coefficient, gain, I²S, top | Thiết kế tổng hợp | Verilog/SystemVerilog, fixed-point |
| Tạo sine/multi-tone, tính reference số thực, đo gain, kiểm tra assertions | **Testbench** (không tổng hợp) | SystemVerilog `real`, `$sin`, `$cos`, `$sqrt`, `$ln` |
| Waveform và log PASS/FAIL | Mô phỏng | Questa/ModelSim (tệp .do dùng để chạy simulator, không phải code DSP) |
| Resource/Fmax/slack | FPGA | Quartus + TimeQuest |

**Không dùng `real`, `$sin`, `$cos` trong RTL tổng hợp.** Các hệ số chạy trên FPGA phải là số nguyên fixed-point, lựa chọn theo `gain_sel`.

## 2. Tạo tín hiệu trong testbench SV

Ví dụ dưới đây **chỉ dành cho testbench**, với `fs=48000`, `amplitude` nhỏ để tránh clipping:

```systemverilog
localparam real PI = 3.141592653589793;
localparam real FS = 48000.0;

// n la so thu tu mau audio, KHONG phai clock cycle 50 MHz.
// Return real trong khoang [-1,1]. Convert sang signed PCM24
// bang $rtoi(...) truoc khi drive DUT.
function automatic real tone(input integer n, input real freq);
    return 0.05 * $sin(2.0 * PI * freq * n / FS);
endfunction

function automatic real multi_tone(input integer n);
    return tone(n,100.0) + tone(n,1000.0) + tone(n,10000.0);
endfunction
```

Khi drive DUT, tạo `input_valid` đúng giao thức: giá trị mẫu giữ ổn định tại cạnh `clk` chấp nhận mẫu. Đọc `sample_out` **chỉ khi `output_valid` được xác nhận**, không so sánh theo số chu kỳ tùy ý.

## 3. Đo gain mà không cần FFT/Python

Có thể đo biên độ của một tone bằng phép **tương quan sine/cosine (DFT tại một tần số)** ngay trong SystemVerilog testbench:

```text
I  = sum(y[n] * cos(2*pi*f*n/fs))   // over N valid output samples
Q  = sum(y[n] * sin(2*pi*f*n/fs))
A  = 2/N * sqrt(I*I + Q*Q)
GdB = 20 * ln(Aout/Ain) / ln(10)
```

- Bỏ các mẫu quá độ sau reset/đổi gain để đo trạng thái xác lập.
- Đảm bảo tone phù hợp cửa sổ đo: ví dụ **N=4800 mẫu** ở fs=48kHz cho 100Hz, 1kHz và 10kHz là các chu kỳ nguyên.
- Đo `Ain` và `Aout` bằng cùng công thức; tránh chia cho 0 và bỏ các tín hiệu clipping khỏi phép đo gain tuyến tính.
- Nếu muốn so sánh reference độc lập, tính trực tiếp đáp ứng `H(e^{jω})` từ hệ số a1,a2,b0,b1,b2 **trong testbench SV**, sau đó so sánh gain RTL đo được.
- Không copy phép tính số nguyên của DUT làm oracle duy nhất; testbench phải phát hiện sai dấu, sai shift, sai coefficient và sai timing.

## 4. Testcase bắt buộc

| Module | Testcase |
|---|---|
| TV1 Bass/Mid | `gain_sel=0..8`, 100/1000/10000Hz, impulse, reset, sát biên PCM, valid |
| TV2 Treble | `gain_sel=0..8`, 100/1000/10000Hz, step gain, clipping, valid |
| TV2 Controller | KEY debounce, tăng/giảm 1 bước, độc lập 3 band, bypass, reset |
| TV3 Audio | I²C ACK, I²S RX/TX framing, 48kHz continuity, CDC, loopback và integration |
| Leader | Nhận log PASS/FAIL, kiểm tra testcase, đối chiếu frequency response, chuẩn bị báo cáo |

**Chú ý throughput:** 50MHz/48kHz = **1041⅔ clock/mẫu**. Testbench cấp dữ liệu theo `input_valid` đã thống nhất; có thể tạo pulse hợp lệ với khoảng cách **1042, 1042, 1041** chu kỳ luân phiên nếu muốn mô phỏng gần đúng 48kHz thực tế. Testbench đơn vị có thể đẩy mẫu nhanh hơn để tiết kiệm thời gian, với điều kiện không vi phạm **initiation interval** của module. Đo tần số theo **chỉ số mẫu**, không theo số chu kỳ clock CPU.

## 5. Tài liệu nộp

- `rtl/**/*.sv` hoặc `rtl/**/*.v`
- `tb/tb_*.sv` self-checking, testcase và log `PASS/FAIL`
- Ảnh waveform nguyên gốc, bảng đo gain từ simulator, điều kiện kiểm thử
- Quartus QPF/QSF/SDC và báo cáo synthesis/timing do TV3 chạy thật

Không bắt buộc chương trình FFT riêng. Chỉ dùng kết quả mà nhóm **đã trực tiếp mô phỏng hoặc đo trên phần cứng**.
