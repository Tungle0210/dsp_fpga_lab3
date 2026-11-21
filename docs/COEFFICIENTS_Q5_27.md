# Biquad EQ – 27 bộ hệ số signed 32-bit Q5.27 (v1.0)

**Quy định sử dụng:** Đây là bảng **hằng số đầu vào** của RTL. TV1/TV2 viết bộ chọn hệ số bằng `case(gain_sel)`/`localparam`/SystemVerilog ROM hoặc `$readmemh` vào ROM tĩnh. Không cần và **không dùng Python/MATLAB** để sinh coefficient khi chạy mô phỏng hoặc trên FPGA. Kiểm tra kết quả bằng SystemVerilog testbench.

> **Hệ số tham chiếu đã tính theo RBJ Audio EQ Cookbook; CHƯA chứng minh khớp RTL hoặc FPGA**. fs=48,000 Hz, a0=1; lượng tử hóa round-to-nearest ties-to-even × 2^27. Công thức RTL: y=b0x0+b1x1+b2x2−a1y1−a2y2. Thứ tự: **B0, B1, B2, A1, A2**.

**gain_sel 0..8** lần lượt là **−12,−9,−6,−3,0,+3,+6,+9,+12 dB**. Literal SystemVerilog hexadecimal two's-complement. **Không được copy nguyên logic Q3.29 từ Lab 2**. Kết quả số ở đây là mô hình tính hệ số, phải chạy HDL testbench và kiểm tra clipping.

| Band | sel | dB | B0 | B1 | B2 | A1 | A2 | max `|pole|` |
|---|---:|---:|---|---|---|---|---|---:|
| Bass | 0 | −12 | `32'sh07DEE2FC` | `32'shF0843D36` | `32'sh079DF035` | `32'shF085D33D` | `32'sh077E6937` | 0.967845 |
| Bass | 1 | −9 | `32'sh07E753B0` | `32'shF0799C91` | `32'sh07A0545F` | `32'shF07AC3B7` | `32'sh0788CF35` | 0.970465 |
| Bass | 2 | −6 | `32'sh07EF9E84` | `32'shF06FDD62` | `32'sh07A206E1` | `32'shF0709DDB` | `32'sh079265DE` | 0.972874 |
| Bass | 3 | −3 | `32'sh07F7D2D3` | `32'shF066EF6C` | `32'sh07A30A7C` | `32'shF0674E75` | `32'sh079B3C58` | 0.975089 |
| Bass | 4 | 0 | `32'sh08000000` | `32'shF05EC3E2` | `32'sh07A360D8` | `32'shF05EC3E2` | `32'sh07A360D8` | 0.977126 |
| Bass | 5 | +3 | `32'sh08083591` | `32'shF0574D56` | `32'sh07A30A83` | `32'shF056EDEB` | `32'sh07AAE0A9` | 0.978998 |
| Bass | 6 | +6 | `32'sh0810834C` | `32'shF0507FB7` | `32'sh07A206EF` | `32'shF04FBDB1` | `32'sh07B1C834` | 0.980718 |
| Bass | 7 | +9 | `32'sh0818F956` | `32'shF04A5049` | `32'sh07A05476` | `32'shF0492589` | `32'sh07B8230D` | 0.982299 |
| Bass | 8 | +12 | `32'sh0821A854` | `32'shF044B59B` | `32'sh079DF057` | `32'shF04318E8` | `32'sh07BDFBF9` | 0.983751 |
| Mid | 0 | −12 | `32'sh071D20D9` | `32'shF27BCF32` | `32'sh0684EB91` | `32'shF27BCF32` | `32'sh05A20C6A` | 0.942325 |
| Mid | 1 | −9 | `32'sh07579360` | `32'shF228AB8D` | `32'sh069E5456` | `32'shF228AB8D` | `32'sh05F5E7B6` | 0.922312 |
| Mid | 2 | −6 | `32'sh07902D98` | `32'shF1DF8EEF` | `32'sh06AF783E` | `32'shF1DF8EEF` | `32'sh063FA5D6` | 0.883786 |
| Mid | 3 | −3 | `32'sh07C7F073` | `32'shF19FA9F2` | `32'sh06B82784` | `32'shF19FA9F2` | `32'sh068017F7` | 0.901413 |
| Mid | 4 | 0 | `32'sh08000000` | `32'shF16820C1` | `32'sh06B81BD7` | `32'shF16820C1` | `32'sh06B81BD7` | 0.916459 |
| Mid | 5 | +3 | `32'sh0839A375` | `32'shF13815D1` | `32'sh06AEED73` | `32'shF13815D1` | `32'sh06E890E8` | 0.929278 |
| Mid | 6 | +6 | `32'sh087647B2` | `32'shF10EB1D2` | `32'sh069C08A4` | `32'shF10EB1D2` | `32'sh07125056` | 0.940182 |
| Mid | 7 | +9 | `32'sh08B78434` | `32'shF0EB2927` | `32'sh067EA34B` | `32'shF0EB2927` | `32'sh0736277E` | 0.949443 |
| Mid | 8 | +12 | `32'sh08FF2282` | `32'shF0CCBF50` | `32'sh0655B202` | `32'shF0CCBF50` | `32'sh0754D485` | 0.957299 |
| Treble | 0 | −12 | `32'sh02960E41` | `32'shFD5C2607` | `32'sh00EFCF17` | `32'shF42C8195` | `32'sh04B581CA` | 0.767220 |
| Treble | 1 | −9 | `32'sh036D052D` | `32'shFC42C086` | `32'sh01569CF6` | `32'shF4878CD6` | `32'sh047ED5D3` | 0.749621 |
| Treble | 2 | −6 | `32'sh048AE98C` | `32'shFABD31F5` | `32'sh01E7E5C6` | `32'shF4E97D1A` | `32'sh0446842D` | 0.731048 |
| Treble | 3 | −3 | `32'sh06070161` | `32'shF8A441E5` | `32'sh02B435E3` | `32'shF552A518` | `32'sh040CD411` | 0.711522 |
| Treble | 4 | 0 | `32'sh08000000` | `32'shF5C35013` | `32'sh03D21BE3` | `32'shF5C35013` | `32'sh03D21BE3` | 0.691080 |
| Treble | 5 | +3 | `32'sh0A9E44EA` | `32'shF1D412A0` | `32'sh05602956` | `32'shF63BBF11` | `32'sh0396C1CF` | 0.669785 |
| Treble | 6 | +6 | `32'sh0E16B969` | `32'shEC791C0B` | `32'sh07878C52` | `32'shF6BC25C8` | `32'sh035B3BFD` | 0.647725 |
| Treble | 7 | +9 | `32'sh12AE72D1` | `32'shE536DC7A` | `32'sh0A7F6849` | `32'shF744A75F` | `32'sh03201035` | 0.625025 |
| Treble | 8 | +12 | `32'sh18BF4617` | `32'shDB6AC3B9` | `32'sh0E911C1B` | `32'shF7D55301` | `32'sh02E5D2EA` | 0.601846 |

**Kiểm tra tính toán (không phải Verilog):** Max |pole| = **0.983751 < 1** sau lượng tử hóa. Ví dụ đáp ứng tại +12dB: Bass 100Hz khoảng **+11.606 dB**, Mid 1kHz **+12.000 dB**, Treble 10kHz **+11.766 dB**.

Nguồn công thức: https://www.w3.org/TR/audio-eq-cookbook/.
