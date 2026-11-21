# Kế thừa Lab 1 và Lab 2: chỉ dẫn cho người phụ trách

## Lab 1 — Repo tham khảo
https://github.com/hominhthao/dsp_fpga_lab1

| Thành phần Lab 1 | Đánh giá cho Lab 3 |
|---|---|
| rtl/common/audio_pll.v | Có thể tham khảo PLL 50→12.288MHz, phải kiểm tra Quartus/IP trên target |
| rtl/common/i2c_master_write.sv | Tham khảo I²C write cho cấu hình WM8731; test ACK/error |
| rtl/common/wm8731_config.sv | **PHẢI SỬA**: Lab 1 ghi WM8731 Power Down reg 0x06 = 0x007, **tắt ADC/Line-In/Mic**; Lab 3 phải bật ADC/Line-In và routing đúng |
| rtl/common/wm8731_i2s_tx.sv | Có I²S TX DAC, dùng mono sample phát L/R; cần kiểm chứng frame/timing |
| rtl/common/top.v | Top mẫu CODEC/PLL/KEY/SW; **không có đủ RX path cho Lab 3** |
| rtl/common/debounce.sv | Có thể tham khảo debouncer KEY |
| rtl/common/sync_bus_2ff.sv | Tham khảo đồng bộ tín hiệu điều khiển, **không dùng độc lập từng bit để truyền PCM đa bit** |
| rtl/common/toggle_sync_pulse.sv | Tham khảo cơ chế xung toggle CDC; phải thiết kế data-hold handshake an toàn |
| quartus/Lab1_DE10_Standard.qsf | Tham khảo pin của CLOCK_50, CODEC DAC, I2C, KEY, SW; **bổ sung AUD_ADCDAT, AUD_ADCLRCK**, xác minh theo manual |
| tb/tb_lab1_top.sv | Tốt để tham khảo mock I²C ACK + I²S receive monitor + self-check + test CDC |

**Không** sao chép nguyên top Lab 1 và coi Lab 3 đã có ADC. Yêu cầu audio input từ thiết bị ngoài là thay đổi kiến trúc quan trọng.

## Lab 2 — ZIP được nhóm cung cấp
- Tham khảo **FIR baseline/folded** và **IIR biquad** về datapath, lựa chọn fixed-point, testbench.
- Kiến trúc nội bộ IIR Lab 2 có **Q4.28** và coefficient **Q3.29**, trong khi Lab 3 đổi sang **Q5.27** để chứa được hệ số high-shelf +12 dB.
- **Không dùng nguyên coefficients hay hàm round-shift Q3.29**, vì đáp ứng cần low-shelf, peaking, high-shelf theo spec mới.
- Phần IIR của Lab 2 có valid/commit đặc thù; nếu dùng lại cần wrapper PCM24, gain_sel, input_valid/output_valid và kiểm tra throughput 48 kHz.

## Quy tắc reuse
- Ghi nguồn gốc module tham khảo trong PR.
- Rà soát quyền sử dụng và chỉnh sửa phù hợp quy định môn học; không sao chép code nhóm khác để nộp mà không được phép.
- Với mỗi module reused: viết lại testbench của Lab 3, chụp chứng cứ và kiểm tra trên phần cứng thực tế.
