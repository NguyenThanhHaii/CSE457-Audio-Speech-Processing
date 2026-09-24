# BÁO CÁO LAB 1 - PHÂN TÍCH VÀ XỬ LÝ TÍN HIỆU ÂM THANH SỐ

**Họ tên:** Nguyễn Thanh Hai  
**MSSV:** 2351260650  
**Môn học:** CSE457 - Xử lý âm thanh và tiếng nói

## 1. Mục tiêu

Thực hành đọc và phân tích tín hiệu âm thanh ở miền thời gian, miền tần số và miền thời gian - tần số. Sau đó thử lọc FIR, lượng tử hóa, resampling và tính bitrate/compression ratio.

## 2. Dữ liệu

File sử dụng: `lab1_input_stereo_44k1.wav`

- Sampling rate: **44,100 Hz**
- Số kênh: **2 (stereo)**
- Bit depth: **16 bit/sample**
- Thời lượng: **12 s**
- Kích thước file: **2.019 MiB**
- Số mẫu mỗi kênh: **529,200**

Sau khi chuyển về mono, RMS của tín hiệu là khoảng **0.3422**.

## 3. Kết quả

### A. Đọc và kiểm tra dữ liệu

File được đọc thành công và chuẩn hóa về miền `[-1, 1]`; giá trị mono thực tế nằm trong khoảng **-0.7190 đến 0.7380**. RMS kênh trái khoảng **0.3824**, kênh phải **0.3576**, mono **0.3422**.

### B. Miền thời gian

- Peak: **0.738022**
- RMS: **0.342194**
- Energy: **61,967.41**
- Không có mẫu nào vượt ngưỡng 0.999 nên chưa có clipping.
- Đoạn 0.5-1.5 s: Peak **0.7380**, RMS **0.4358**, Energy **8,376.57**.
- Đoạn 6-7 s: Peak **0.6740**, RMS **0.1861**, Energy **1,527.11**.

### C. FFT

- `NFFT = 65536` → \(\Delta f \approx 0.673\) Hz.
- `NFFT = 131072` → \(\Delta f \approx 0.336\) Hz.
- Các peak chính nằm gần **220.04, 440.08, 660.13 và 879.83 Hz**.

Tăng NFFT làm các bin tần số dày hơn, nhưng không tự làm tăng true resolution nếu độ dài frame không đổi.

### D. STFT và Spectrogram

So sánh frame 10, 25 và 50 ms với hop 10 ms:

- 10 ms: theo dõi thay đổi thời gian tốt hơn.
- 50 ms: biểu diễn tần số rõ hơn.
- 25 ms: cân bằng giữa hai loại độ phân giải.

### E. Window

Rectangular có spectral leakage lớn hơn. Hamming giảm side-lobe và leakage nhưng main-lobe rộng hơn.

### F. Lọc FIR

Đã thực hiện:

- Low-pass 2 kHz.
- High-pass 1 kHz.
- 201 taps, Hamming window.

Group delay xấp xỉ **100 mẫu = 2.268 ms**. Phổ sau lọc thay đổi đúng theo đáp ứng của từng bộ lọc.

### G. Quantization, Resampling và Coding

**Quantization:**

|    Bit |      SNR |
| -----: | -------: |
|  4 bit | 18.42 dB |
|  8 bit | 43.48 dB |
| 16 bit | 91.78 dB |

Số bit càng lớn thì sai số lượng tử càng nhỏ và SNR càng cao.

**Resampling:**

- 16 kHz → Nyquist 8 kHz, 192,000 mẫu.
- 8 kHz → Nyquist 4 kHz, 96,000 mẫu.
- Thời lượng vẫn giữ 12 s.

**Coding:**

PCM 16-bit stereo 44.1 kHz có bitrate **1411.2 kbps** và kích thước lý thuyết cho 12 s khoảng **2.117 MB**:

\[
44100 \times 16 \times 2 = 1411.2\text{ kbps}
\]

Với MP3 128 kbps, kích thước lý thuyết khoảng **0.192 MB**, tiết kiệm khoảng **90.93%**, compression ratio khoảng:

\[
1411.2 / 128 \approx 11.03:1
\]

## 4. Trả lời câu hỏi báo cáo

**1. Vì sao 44.1 kHz chỉ biểu diễn độc lập đến 22.05 kHz?**  
Theo Nyquist, tần số lớn nhất cần nhỏ hơn hoặc bằng \(F_s/2\). Với 44.1 kHz thì Nyquist là 22.05 kHz.

**2. Tăng NFFT từ 2048 lên 8192 nhưng frame vẫn 25 ms thì sao?**  
Khoảng cách bin \(\Delta f\) nhỏ hơn, phổ nhìn mịn hơn. Tuy nhiên lượng thông tin trong frame không tăng nên true resolution không tự tăng.

**3. Vì sao Hamming giảm leakage nhưng có thể làm peak gần nhau khó tách?**  
Hamming giảm side-lobe nhưng làm main-lobe rộng hơn.

**4. FIR 201 taps tại 44.1 kHz trễ bao nhiêu?**  
\((201-1)/2 = 100\) mẫu, tương đương khoảng **2.27 ms**.

**5. Số bit và RMS ảnh hưởng SNR lượng tử thế nào?**  
Tăng số bit làm bước lượng tử nhỏ hơn nên SNR tăng. Nếu tín hiệu có RMS quá thấp so với full-scale thì SNR lượng tử thường giảm.

**6. WAV 16-bit stereo 44.1 kHz dài 60 s có kích thước bao nhiêu?**  
Khoảng **10.584 MB** theo MB thập phân. MP3 128 kbps dài 60 s khoảng **0.96 MB**, tỷ lệ khoảng **11.03:1**.

**7. Khi nào nghe tốt hơn nhưng SNR không lớn hơn?**  
Ví dụ lọc bỏ tiếng hiss/rumble hoặc mã hóa perceptual như MP3. Chất lượng cảm nhận có thể tốt dù waveform khác tín hiệu gốc.

## 5. Kết luận

Qua Lab 1, em đã thực hiện được các bước cơ bản để đọc, phân tích và xử lý tín hiệu âm thanh số bằng Python. Kết quả giúp hiểu rõ hơn mối liên hệ giữa waveform, FFT/STFT, window, filter, quantization và sampling rate.
