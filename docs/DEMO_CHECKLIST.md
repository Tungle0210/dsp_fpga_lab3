# Demo checklist — DE10-Standard

**Chưa nghiệm thu trên phần cứng.**

- [ ] Đúng board 5CSXFC6D6F31C6N, pin ADC/DAC, I²C, KEY, SW.
- [ ] PLL lock, CODEC I²C ACK, ADC/Line-In active, DAC output active.
- [ ] Loopback Line-In→Line-Out có âm, không mất tín hiệu.
- [ ] Switch chọn đúng Bass/Mid/Treble.
- [ ] KEY UP/DOWN làm đúng 1 bước 3 dB cho band đang chọn.
- [ ] Bypass và all 0dB hoạt động, không treo.
- [ ] Phát real audio đa dải, thay gain từng band có tác động nghe/đo được.
- [ ] Thử nguồn đủ nhỏ để không clipping, sau đó cố tình test clipping và chứng minh saturation.
- [ ] Audio xử lý không đứt đoạn khi bấm KEY/đổi band.
- [ ] Lưu waveform, FFT, video quay kit và loa; ghi điều kiện kiểm thử.
- [ ] Quartus/TimeQuest Fmax, slack, resource report; mô tả bottleneck.
- [ ] Báo cáo và ZIP nộp LMS (theo hạn thực tế).

**An toàn audio:** Dùng headphone/loa có volume thấp khi bắt đầu, kiểm tra line-level tại Line-In. Không đấu line-out công suất của amplifier vào CODEC input.
