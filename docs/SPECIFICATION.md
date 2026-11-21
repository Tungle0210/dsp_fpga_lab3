# Lab 3 – System Specification v1.0

> **Đã chốt làm chuẩn phát triển** · **Chưa được xác nhận bằng mô phỏng RTL/hardware** · Khi thay đổi phải cập nhật file này bằng PR do Leader duyệt.

**Giới hạn công cụ:** Viết **RTL + testbench 100% Verilog/SystemVerilog**, Quartus để tổng hợp, Questa/ModelSim để mô phỏng. **Không dùng Python/MATLAB**. Bảng hệ số Q5.27 đã cung cấp được mô tả bằng hằng số Verilog/SystemVerilog (`case`, `localparam` hoặc ROM `.hex` nếu cần). Dùng `$sin`, `$cos`, `$ln`, `real` **chỉ trong testbench**, tuyệt đối không dùng trong RTL tổng hợp.

## A. Thông số hệ thống

| Hạng mục | Giá trị |
|---|---|
| Board / FPGA | DE10-Standard / Cyclone V SX 5CSXFC6D6F31C6N |
| System clock | 50 MHz |
| Audio sampling | 48,000 samples/second |
| Master clock | 12.288 MHz (256 × fs), sử dụng PLL |
| BCLK | 3.072 MHz (64 × fs) |
| I²S | Stereo frame, 32-bit/slot, dữ liệu 24-bit; xác minh timing pha với CODEC |
| Analog Input | WM8731 Line-In/ADC (KHÔNG dùng nhầm cấu hình Lab 1 vốn tắt ADC) |
| Analog Output | WM8731 DAC/Line-Out → loa active/amp |
| Audio v1 | Nhận kênh L, xử lý mono; phát kết quả sang DAC L và R |
| PCM data ports | logic signed [23:0], signed two's complement |
| Reset | rst_n active-low từ KEY[0] |
| Throughput tối thiểu | 1 sample / 20.833333 μs, không mất/đảo mẫu |
| Timing mục tiêu | Fmax ≥ 50 MHz; mọi đường có clock/SDC hợp lệ; slack ≥ 0 |
| DSP processing latency mục tiêu | ≤200 chu kỳ 50 MHz mỗi mẫu (mục tiêu, không phải kết quả đo) |

### Kiến trúc được chốt

**IIR biquad 3 tầng cascade**: Bass Low-Shelf → Mid Peaking → Treble High-Shelf.

| Band | Filter | f0 | Additional parameter |
|---|---|---:|---|
| Bass | Low-Shelf bậc 2 | 250 Hz | Shelf slope S=1 |
| Mid | Peaking EQ bậc 2 | 1,000 Hz | Q=0.75 |
| Treble | High-Shelf bậc 2 | 4,000 Hz | Shelf slope S=1 |

**Chú ý:** f0 của shelving/peaking là tần số đặc trưng, không phải các vách cắt của ba band-pass lý tưởng. Kiến trúc này khác hình FIR ba nhánh tham khảo trong đề nhưng phù hợp quyền chọn IIR.

### Gain mapping

| gain_sel | Gain (dB) |
|---:|---:|
| 0 | −12 |
| 1 | −9 |
| 2 | −6 |
| 3 | −3 |
| 4 | 0 |
| 5 | +3 |
| 6 | +6 |
| 7 | +9 |
| 8 | +12 |

Ba thanh ghi gain_bass, gain_mid, gain_treble **độc lập**, reset đều = 4. Với mỗi band và gain_sel có một bộ 5 hệ số (b0,b1,b2,a1,a2). **Cập nhật hệ số nguyên bộ ở ranh giới sample**, không đổi từng hệ số giữa MACs. Bypass tại top-level.

### Định dạng fixed-point

| Đại lượng | Format |
|---|---|
| Audio input/output | signed PCM 24-bit |
| Sample/internal biquad state | signed 32-bit Q4.28 |
| Biquad coeffs | signed 32-bit Q5.27 |
| Product | signed 64-bit (Q9.55 trước khi scale) |
| Product scale to Q4.28 | Round + arithmetic shift 27 |
| Internal→PCM24 | Round + arithmetic shift 5, saturate signed24 |
| Difference equation | y[n]=b0*x[n]+b1*x[n−1]+b2*x[n−2]−a1*y[n−1]−a2*y[n−2] (a0=1) |

**Bắt buộc:** xác định điểm làm tròn ở datapath, thiết kế guard bits cho tổng tích và state, kiểm soát overflow/saturation. Không mặc định mọi đường 32-bit đủ headroom cho boost 3 band. Nếu lựa chọn saturate ở từng tầng, phải mô tả trade-off và kiểm thử clipping. Hệ số được tạo mới ở [COEFFICIENTS_Q5_27.md](COEFFICIENTS_Q5_27.md).

Không dùng lại trực tiếp hệ số **Q3.29** và phép dịch 29 từ Lab 2 vì Q5.27 là lựa chọn mới. 0 dB tại mỗi band phải gần unity; bypass cho phép so sánh bit-exact sau căn chỉnh latency.

### Chuẩn module từng band

```systemverilog
module bass_eq (
  input  logic               clk,
  input  logic               rst_n,
  input  logic               input_valid,
  input  logic signed [23:0] sample_in,
  input  logic [3:0]         gain_sel,
  output logic               output_valid,
  output logic signed [23:0] sample_out
);
```

Các module mid_eq và treble_eq dùng **đúng port names/types**; khác duy nhất là tên module. **Một input_valid được chấp nhận phải tạo đúng một output_valid** theo thứ tự. Mỗi module phải khai báo latency, initiation interval và hỗ trợ dòng 48 kHz. Nếu sử dụng busy/ready nội bộ phải có wrapper hoặc cơ chế buffer không mất mẫu ở top.

Các state IIR chỉ tiến triển khi có mẫu thật; reset khởi tạo state bằng 0. Từ file coefficients chọn gain_sel 0..8, ngoài miền phải xử lý an toàn (quy về 4 hoặc giữ gain hợp lệ).

### Control mapping

| Hardware | Chức năng |
|---|---|
| KEY[0] | Reset active-low |
| SW[1:0] | 00 Bass / 01 Mid / 10 Treble / 11 no selection |
| KEY[1] | Tăng 3 dB (bão hòa ở +12) |
| KEY[2] | Giảm 3 dB (bão hòa ở −12) |
| KEY[3] | Toggle bypass |
| LEDR | Leader + TV3 chốt bit map trong top, hiển thị selected band, codec_ready, clip/overrun |

KEY phải debounce, phát **một pulse cho mỗi lần nhấn**; cấu hình gợi ý 20 ms = 1,000,000 clock tại 50 MHz. SW đồng bộ vào clock; lưu ý debounce/synchronizer bus không thay thế bảo đảm CDC cho dữ liệu audio nhiều bit.

## B. Đường âm thanh

Audio clock 12.288 MHz / codec interface là clock domain khác 50 MHz. **Không nối trực tiếp bus PCM từ một domain sang domain khác mà không có data-hold handshake hoặc async FIFO hợp lệ**. I²S RX/TX phải tuân thủ frame/channel/bit alignment thực tế. Audio CODEC Lab 1 dùng cấu hình tắt ADC, phải **sửa lại ADC enable và analog input path**.

### Demo/hardware

1. Nạp project ADC→DAC loopback trước, audio phải truyền ổn định.
2. Ghép ba tầng với gain=0 dB, đo đáp ứng và so với bypass.
3. Điều chỉnh độc lập Bass, Mid, Treble tại input có đủ 3 dải; record video.
4. Thử thao tác nhấn liên tục, clipping, reset, thay đổi gain, xác nhận không treo hệ thống.

## C. Nguồn và kiểm chứng

- Công thức thiết kế: https://www.w3.org/TR/audio-eq-cookbook/
- Lab 1: https://github.com/hominhthao/dsp_fpga_lab1
- Lab 2: ZIP tham khảo người dùng cung cấp.
- Lab 3 assignment 7 trang do nhóm cung cấp.

**Kết quả coefficient tính toán không tương đương kết quả Verilog/Questa hay FPGA.** Không tự điền Fmax/ALM/SLACK trước khi chạy Quartus.
