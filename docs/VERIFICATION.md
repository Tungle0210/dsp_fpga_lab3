# Verification & Evidence Plan

> **Chưa có kết quả PASS.** Mọi kết quả phải đi kèm testbench/log/ảnh/CSV; không suy ra đúng từ việc compile thành công.

## 1. Test tại mức module
- **Impulse:** trạng thái IIR đúng, reset/xóa state, latency xác định.
- **9 mức gain:** kiểm từng band ở −12/−9/−6/−3/0/+3/+6/+9/+12 dB.
- **Single-tone:** 100 Hz (Bass), 1 kHz (Mid), 10 kHz (Treble), đo FFT sau thời gian quá độ.
- **Multi-tone:** ví dụ x[n]=0.05·(sin(2π100n/fs)+sin(2π1000n/fs)+sin(2π10000n/fs)).
- **Sweep:** đáp ứng frequency response, nhất là vùng giao nhau 250Hz và 4kHz.
- **Fixed point:** PCM signed min/max, zero, saturation, coefficient update, gain step, no wrap-around.
- **Throughput:** 48k samples/s, input_valid→output_valid 1:1 theo đúng thứ tự.
- **Band independence:** thay 1 gain, 2 gain_sel khác giữ nguyên; đo tác động tần số ngoài band.
- **0 dB:** filter ở unity so với reference; không nhầm gain=0 với mute.

## 2. Mức hệ thống
1. **Loopback ADC→DAC trên DE10**: đo/audio real, không mất mẫu trong 5s.
2. **EQ tất cả 0dB + bypass**: đối chiếu PCM sau căn chỉnh latency; tránh khẳng định bit-exact nếu vẫn qua fixed-point filter.
3. **Tăng/cắt từng band trong khi phát**: ghi video thao tác và kết quả âm thanh rõ ràng.
4. **Reset giữa hoạt động, thay đổi KEY liên tục, clip, chuyển bypass**, không treo/rè bất thường.
5. **Quartus:** compiler report, ALM/register/DSP/RAM, TimeQuest, Fmax, slack, cảnh báo CDC.

## 3. Ngưỡng nghiệm thu gợi ý
- Đo gain ở tone trung tâm (sau khi bỏ quá độ): sai lệch với **mô hình số tham chiếu đã lượng tử hóa hệ số** ≤ 1 dB cho các trường hợp test không clipping.
- Với gain=0 dB tất cả band, đáp ứng phải xấp xỉ phẳng; mô tả sai số số học bằng reference.
- Không được mất/lặp/đảo mẫu trong chuỗi kiểm thử liên tục.
- TimeQuest: ≥ 50 MHz ở miền DSP, **slack ≥ 0** cho các ràng buộc hợp lệ.
- Phải báo cáo các trường hợp clipping và điều kiện headroom; không lấy tín hiệu đã clipping làm bằng chứng đáp ứng filter.

## 4. Quy định nộp evidence
- Đặt file theo dạng: tv1_bass_100hz_plus12_fft.png, tv2_gain_reset_wave.png, tv3_loopback_DE10.mp4.
- Mỗi ảnh có **tên tín hiệu, trục thời gian/tần số, điều kiện test, giá trị đặt, kết quả**.
- Không cắt/chỉnh sửa ảnh theo cách làm mất thông tin đo. Lưu ảnh gốc + ảnh chú thích nếu cần.
- Log phải chứa PASS/FAIL, số test, môi trường simulator và phiên bản mã/commit.
- Báo cáo phải phân biệt **dữ liệu Python reference**, **HDL simulation**, **hardware measurement**.

## 5. Reporting
Báo cáo PDF + RTL + testbench + project Quartus đóng thành 1 ZIP theo đề, deadline lấy LMS. Các phần: cơ sở DSP, lựa chọn filter, block diagram/data path, Q-format, testbench & FFT, synthesis/time, demo, giới hạn và hướng nâng cấp.
