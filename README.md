# EE3041 – DSP on FPGA | LAB 3: AUDIO EQUALIZER

**Nhóm Lab 3 – DE10-Standard | Leader: quản lý specification, code review, QA và báo cáo**

> **Trạng thái: BẮT ĐẦU DỰ ÁN (Planning / Development).** README này là hướng dẫn nhóm, **không có nghĩa RTL, mô phỏng hoặc demo đã PASS**. Đọc kỹ các tài liệu liên kết trước khi viết code.

---

## 1. TỔNG QUAN VÀ YÊU CẦU ĐỀ BÀI

**Mục tiêu:** Thiết kế **Audio Equalizer 3 dải Bass – Mid – Treble** trên FPGA, điều chỉnh gain từng dải độc lập theo thời gian thực, nhận âm thanh từ laptop/điện thoại và phát âm thanh đã xử lý ra loa.

Theo đề Lab 3:
- Tự chọn, giải thích thiết kế **FIR hoặc IIR**; ít nhất **3-band**.
- Viết **HDL và testbench riêng**; kiểm chứng tín hiệu có nhiều thành phần tần số và audio sample.
- Thực hiện **synthesis/TimeQuest**, phân tích resources, Fmax và timing.
- **Demo thực tế**: nguồn âm thanh → FPGA → loa; điều chỉnh gain từng band.
- Bonus: nhiều band hơn, điều khiển bằng PC, tham số hóa, phân tích nâng cao… **chỉ làm sau khi 3-band hoạt động ổn định**.

**Rubric (10 điểm cơ bản):** EQ understanding 0.5 · Filter choice/spec 0.5 · Working implementation 2 · Filter verification 2 · Synthesis 2 · Real-time gain demo **3**.

---

## 2. KIẾN TRÚC NHÓM ĐÃ CHỐT — SPEC v1.0

```text
Laptop / Phone (Line-Out)
           |
           v
  WM8731 Line-In / ADC / I2S RX             KEY / SW
           |                                    |
       PCM24 / valid                            v
           |                           Gain Controller [TV2]
           v                                    |
    Bass Low-Shelf IIR [TV1] <----- gain_bass ---+
           |
           v
    Mid Peaking IIR [TV1]   <------ gain_mid ----+
           |
           v
    Treble High-Shelf IIR [TV2] <- gain_treble --+
           |
       Round / Saturation
           |
         Bypass MUX
           |
           v
   I2S TX / WM8731 DAC / Line-Out --> Active Speaker
           ^
     CODEC + TOP + CDC [TV3]
```

**Chọn IIR biquad 3 tầng NỐI TIẾP**, không phải FIR 3 nhánh song song ở sơ đồ ví dụ trong đề. Đề cho phép IIR. Mỗi band có gain riêng bằng bộ hệ số tương ứng; thứ tự ghép ở top-level: **Bass → Mid → Treble**.

**Điểm quan trọng:** TV1 và TV2 **làm việc độc lập ở cấp module**; không instance module của nhau. TV3 mới tích hợp ba tầng. Gain 0 dB tương ứng **gain_sel=4** ở mỗi band, không phải tắt filter.

| Thông số | Giá trị chốt để code |
|---|---|
| FPGA | **DE10-Standard**, Cyclone V SX 5CSXFC6D6F31C6N |
| DSP clock | **50 MHz** |
| Audio | **48 kHz**, MCLK **12.288 MHz**, BCLK **3.072 MHz**, I²S 2×32 bit slot |
| ADC/DAC interface | **Signed PCM 24-bit**; v1 xử lý **mono L**, phát giống nhau ở L/R |
| Bass | **Low-Shelf biquad**, f0 = **250 Hz**, slope S=1 |
| Mid | **Peaking biquad**, f0 = **1 kHz**, Q=**0.75** |
| Treble | **High-Shelf biquad**, f0 = **4 kHz**, slope S=1 |
| Gain từng band | **−12 đến +12 dB**, bước **3 dB**, tổng cộng **9 mức** |
| Gain selector | **4-bit unsigned (0..8)**; **4 ↔ 0 dB**, sau reset = 4 |
| Nội bộ IIR | **32-bit signed Q4.28** |
| Hệ số | **32-bit signed Q5.27** (5 hệ số/biquad, 27 bộ cho 3 dải × 9 gain) |
| DSP throughput | Ít nhất **1 mẫu/20.833 µs**, không mất mẫu |
| Timing | Target **Fmax ≥ 50 MHz**, slack hợp lệ ≥ 0 ns; **chưa có số liệu thực** |

**Tài liệu bắt buộc:** [Specification v1.0](docs/SPECIFICATION.md) · [Bảng coefficients Q5.27](docs/COEFFICIENTS_Q5_27.md).

---

## 3. PHÂN CÔNG NHÓM — TV1 VÀ TV2 ĐỘC LẬP

### TV1 – Bass & Mid Equalizer
**Thư mục:** rtl/bass_mid/, coeff/, tb/, doc/simulation/

- Thiết kế **bass_eq.sv** (low-shelf) và **mid_eq.sv** (peaking), dùng spec và hệ số đã chốt.
- Viết **tb_bass_eq.sv**, **tb_mid_eq.sv** (tự kiểm tra PASS/FAIL).
- Test gain 0, ±3, ±6, ±9, ±12 dB; impulse, single-tone, multi-tone, reset, giới hạn dữ liệu.
- Nộp: **RTL + testbench + log PASS + FFT/frequency-response + latency + tài liệu Q-format**.
- **Không phụ thuộc TV2**; module nhận PCM24 mẫu giả lập, gain_sel đầu vào.

### TV2 – Treble & Gain Controller
**Thư mục:** rtl/treble_control/, coeff/, tb/, doc/simulation/

- Thiết kế **treble_eq.sv** (high-shelf).
- Thiết kế **gain_controller.sv**: KEY/SW, debounce, 3 gain_sel độc lập, giữ giới hạn 0..8, bypass.
- Test **treble_eq** và **gain_controller** bằng các nguồn giả lập; **không import TV1**.
- Nộp: **RTL + testbench + log PASS + FFT/frequency-response Treble + bảng KEY/SW + latency**.

### TV3 – CODEC, FPGA & Integration
**Thư mục:** rtl/audio/, rtl/common/, rtl/top/, quartus/, tb/, doc/

- Tái kiểm tra CODEC/PLL/I²C/I²S TX từ Lab 1; **bổ sung ADC + I²S RX** (Lab 1 đã **tắt ADC**).
- Tạo **loopback Line-In → Line-Out** chạy trên board trước khi tích hợp EQ.
- Viết **equalizer_top.sv** nối Bass → Mid → Treble, đảm bảo sample-valid, CDC, reset, pipeline/timing, bypass.
- Chạy Quartus, gán pin DE10, SDC, TimeQuest, xuất **.sof, utilization, Fmax, slack**.
- Nộp: **top-level + CODEC + testbench tích hợp + Quartus project + ảnh/video demo thật**.

### LEADER – Spec, QA, Report, Demo
- Giữ nguyên bản spec thống nhất; duyệt thay đổi kỹ thuật có ảnh hưởng cả nhóm.
- Quản lý GitHub Issues, PR, deadline; kiểm tra RTL + testbench của từng người.
- Duyệt mô hình tham chiếu, độ chính xác, rounding/saturation, clipping, timing.
- Chủ trì ghép hệ thống, tổ chức demo, viết báo cáo và ZIP nộp LMS.
- **Không ghi PASS nếu chưa có bằng chứng chạy thực tế**.

**Xem đầu ra và tiêu chí nghiệm thu:** [Phân công chi tiết](docs/TASKS.md) · [Test Plan](docs/VERIFICATION.md).

---

## 4. CHUẨN GIAO TIẾP CHUNG

Để ghép các khối độc lập, module EQ nhận **signed PCM24**, cùng miền clock DSP 50 MHz, reset active-low, và dùng giao thức **input_valid → output_valid**. Mỗi module nhận đúng 1 mẫu khi input_valid = 1, xuất đúng 1 output_valid theo thứ tự, có latency công bố.

```systemverilog
module example_eq (
    input  logic               clk,
    input  logic               rst_n,
    input  logic               input_valid,
    input  logic signed [23:0] sample_in,
    input  logic        [3:0]  gain_sel,
    output logic               output_valid,
    output logic signed [23:0] sample_out
);
```

**Đây chỉ là interface (không phải RTL hoàn chỉnh).** Tất cả module phải hỗ trợ 48 kHz liên tục; nếu module cần nhiều chu kỳ, ghi rõ latency, busy/ready hoặc cơ chế chấp nhận mẫu sao cho TV3 ghép đúng. Không đổi tên port/Q-format tùy ý. Biquad chỉ đổi **đồng bộ trọn bộ 5 coefficients ở ranh giới mẫu**, tránh lệch hệ số trong một phép tính.

---

## 5. GIT WORKFLOW — BẮT BUỘC

1. Clone repo và đọc README + SPECIFICATION + TASKS.
2. Tạo branch riêng **feat/tv1-bass-mid**, **feat/tv2-treble-control** hoặc **feat/tv3-audio-integration**.
3. Viết RTL + self-checking testbench; chỉ mở PR khi đã chạy test và lưu evidence.
4. Commit rõ mục đích: feat(...), fix(...), test(...), docs(...). **Không push trực tiếp main**.
5. Leader review → merge PR → mọi người pull lại main.

```bash
git clone https://github.com/Tungle0210/dsp_fpga_lab3.git
cd dsp_fpga_lab3
git switch -c feat/tv1-bass-mid
# ... code, chạy testbench, lưu log ...
git add .
git commit -m "feat(bass): add low-shelf biquad and testbench"
git push -u origin feat/tv1-bass-mid
```

Có thể làm trên máy cá nhân/EDA Playground/Questa, nhưng kết quả cuối phải tương thích Quartus trên DE10-Standard. Xem [Hướng dẫn làm việc](docs/WORKFLOW.md).

---

## 6. CẤU TRÚC REPOSITORY

```text
dsp_fpga_lab3/
├── README.md
├── docs/
│   ├── SPECIFICATION.md            # Spec kỹ thuật v1.0
│   ├── COEFFICIENTS_Q5_27.md       # Bảng hệ số 27 bộ
│   ├── TASKS.md                     # Task & nghiệm thu từng TV
│   ├── VERIFICATION.md              # Test plan + evidence
│   ├── REUSE_LAB1_LAB2.md           # Module có thể kế thừa
│   └── WORKFLOW.md                  # Git/PR & cách nộp
├── rtl/
│   ├── bass_mid/                    # TV1
│   ├── treble_control/              # TV2
│   ├── audio/                       # TV3
│   ├── common/                      # TV3
│   └── top/                         # TV3
├── coeff/                           # Coefficients / .hex khi có
├── python/                          # Mô hình tham chiếu khi có
├── tb/                              # Các testbench
├── quartus/                         # qpf/qsf/sdc; không push cache
└── doc/
    ├── simulation/                  # waveform, FFT, log
    ├── synthesis/                   # resource, timing, Fmax
    └── hardware_demo/               # ảnh/video demo
```

---

## 7. CHECKLIST & MỐC NGHIỆM THU

| Mốc | Điều kiện | Phụ trách |
|---|---|---|
| M1 | Spec + giao diện + mapping được thống nhất | Leader |
| M2 | Bass/Mid RTL + TB/FFT PASS | TV1 |
| M3 | Treble/Gain Controller + TB PASS | TV2 |
| M4 | ADC→DAC loopback trên DE10 PASS | TV3 |
| M5 | Tích hợp 3-band, gain độc lập, unity/bypass, không mất mẫu | TV3 + Leader |
| M6 | Quartus compile + phân tích tài nguyên/Fmax/slack | TV3 |
| M7 | **Demo audio thật + điều chỉnh real-time** | Cả nhóm |
| M8 | Báo cáo PDF + toàn bộ source nộp **1 ZIP** trên LMS | Leader |

**Hiện các mốc M2–M8 CHƯA có kết quả PASS.**

---

## 8. THAM KHẢO

- Đề chính thức: **EE3041 DSP on FPGA – Lab 3: Audio Equalizer** (PDF 7 trang do nhóm cung cấp; không đăng lại công khai nếu chưa có quyền).
- [Repo Lab 1 – leader trước](https://github.com/hominhthao/dsp_fpga_lab1) · Lab 2: ZIP nguồn do nhóm cung cấp.
- [W3C Audio EQ Cookbook](https://www.w3.org/TR/audio-eq-cookbook/) – công thức biquad low-shelf, peaking, high-shelf.
- [Bảng kế thừa & lưu ý về CODEC Lab 1 và FIR/IIR Lab 2](docs/REUSE_LAB1_LAB2.md).

> **Nguyên tắc:** Tài liệu, mã RTL và bài báo cáo do nhóm thực hiện; tham khảo mã cũ phải rà soát bản quyền và kiểm thử lại. **Không công bố hình, thông số đo hoặc kết quả FPGA chưa thực hiện.**
