# CSE457 • XỬ LÝ ÂM THANH VÀ TIẾNG NÓI — BÀI THỰC HÀNH LAB 3
## NHẬN DẠNG TIẾNG NÓI NÂNG CAO VỚI HMM (HIDDEN MARKOV MODEL)

**Trường Đại học Thủy Lợi (TLU) — Khoa Công nghệ Thông tin — Bộ môn Trí tuệ Nhân tạo**  
- **Sinh viên thực hiện**: Lê Thị Thùy Linh  
- **Mã sinh viên**: 2351260662  
- **Lớp**: 65TTNT  
- **Học phần**: CSE457 - Xử lý Âm thanh và Tiếng nói  

---

## 1. Cấu trúc thư mục dự án đầy đủ (Đáp ứng 100% Mục 7 - Lab 3.pdf)

```text
Lab03_2351260662_LeThiThuyLinh/
├── Lab 3.pdf                  # Đề bài thực hành chính thức của Bộ môn
├── Lab03.ipynb                # [SẢN PHẨM CHÍNH 1] Jupyter Notebook hoàn chỉnh, có chú thích toán học LaTeX chi tiết
├── BaoCao_Lab03.pdf          # [SẢN PHẨM CHÍNH 2] Báo cáo học thuật định dạng PDF (Font Times New Roman 12pt chuẩn)
├── BaoCao_Lab03.docx         # Báo cáo học thuật định dạng Microsoft Word
├── lexicon.txt                # Từ điển phát âm (Pronunciation Lexicon) cho 5 lệnh tiếng Việt (format ASR)
├── corpus.txt                 # Tập ngữ liệu các câu lệnh điều khiển hệ thống phục vụ Bigram LM
├── bigram_counts.json         # Thống kê tần suất Unigram/Bigram và xác suất chuyển từ có làm mịn Add-k
├── README.md                  # Hướng dẫn chi tiết cách chạy và cấu trúc toàn bộ sản phẩm nộp
├── generate_dataset.py        # Script tổng hợp dữ liệu khẩu ngữ chuẩn 16kHz mono 16-bit PCM
├── generate_lab03_ipynb.py    # Builder script tạo file notebook
├── export_artifacts.py        # Script tự động xuất models/, figures/, tables và docx
│
├── models/                    # Thư mục lưu trữ các mô hình HMM đã huấn luyện (Serialized Models)
│   ├── mo.pkl                 # Model HMM từ "mở"
│   ├── dong.pkl               # Model HMM từ "đóng"
│   ├── len.pkl                # Model HMM từ "lên"
│   ├── xuong.pkl              # Model HMM từ "xuống"
│   ├── dung.pkl               # Model HMM từ "dừng"
│   └── all_models.pkl         # Toàn bộ dictionary chứa cả 5 mô hình
│
├── figures/                   # Thư mục chứa các biểu đồ, ma trận nhầm lẫn và bảng điểm CSV
│   ├── confusion_matrix.png          # Ma trận nhầm lẫn trên tập Test sạch (Accuracy = 100%)
│   ├── viterbi_alignment_mo.png      # Biểu đồ Viterbi Trellis Alignment từ "mở"
│   ├── viterbi_alignment_dong.png    # Biểu đồ Viterbi Trellis Alignment từ "đóng"
│   ├── heatmap_mfcc_dynamic.png      # Đối sánh Heatmap 39 đặc trưng (MFCC, Delta, Delta-Delta)
│   ├── likelihood_scores.csv         # Bảng điểm acoustic likelihood đối với 5 model trên các test samples
│   └── dtw_vs_hmm_comparison.csv    # Bảng đối sánh đa tiêu chí giữa DTW và HMM
│
└── dataset/                   # Bộ dữ liệu 5 lệnh tiếng Việt (58 files)
    ├── train/                 # 30 files huấn luyện (6 files/lớp: mo, dong, len, xuong, dung)
    ├── test/                  # 10 files kiểm tra sạch (2 files/lớp: mo, dong, len, xuong, dung)
    └── test_mismatch/         # 5 files kiểm tra trong điều kiện tạp âm & lệch giọng nữ (Robustness E5)
```

---

## 2. Hướng dẫn cài đặt môi trường

Cài đặt các gói thư viện cần thiết bằng lệnh:
```bash
pip install numpy scipy matplotlib librosa soundfile scikit-learn hmmlearn python-docx
```

---

## 3. Hướng dẫn chạy chương trình với Jupyter Notebook

1. Mở file [Lab03.ipynb](file:///d:/Lab03_2351260662_LeThiThuyLinh/Lab03.ipynb) trong Visual Studio Code hoặc Jupyter Lab / Jupyter Notebook.
2. Chọn Kernel Python 3 phù hợp (đã cài các thư viện `librosa`, `hmmlearn`, `scikit-learn`, `matplotlib`).
3. Nhấn **"Run All"** để thực thi toàn bộ pipeline từ đầu đến cuối, quan sát trực tiếp các biểu đồ Heatmap phổ, Trellis Viterbi Alignment, ma trận nhầm lẫn Confusion Matrix và các bảng so sánh.

---

## 4. Bảng đối chiếu các tiêu chí chấm điểm theo Rubric 10 điểm (Mục 7.2)

| Hạng mục Rubric | Điểm tối đa | Mức đạt & Bằng chứng thực nghiệm |
| :--- | :---: | :--- |
| **Chuẩn bị dữ liệu + MFCC/Δ/Δ² đúng** | **1.5** | Đọc WAV 16kHz mono, cắt silence VAD (`top_db=30`), pre-emphasis 0.97, trích xuất 13 MFCC tĩnh + 13 Delta + 13 Delta-Delta ghép thành ma trận $T \times 39$, chuẩn hóa utterance-level CMVN. Đúng phân chia train/test (6 train + 2 test/lớp), không rò rỉ dữ liệu (leakage). |
| **Hiểu & cấu hình HMM** | **2.0** | Cấu hình Bakis topology Trái-sang-Phải 5 trạng thái: $\pi=[1, 0, 0, 0, 0]$, $a_{ii}=0.7$, $a_{i, i+1}=0.3$, $a_{NN}=1.0$. Phân bố Gaussian Diagonal Covariance. Giải thích cặn kẽ tại sao Diagonal Covariance phù hợp hơn Full Covariance khi $D=39$ và dữ liệu nhỏ. |
| **Huấn luyện + likelihood + Viterbi** | **2.0** | Ghép chuỗi huấn luyện $X\_concat$ kèm danh sách `lengths`. Cả 5 model hội tụ tốt (`converged = True`). Hàm `recognize()` chọn $\arg\max \log P(X\|\lambda_w)$. Gióng hàng Viterbi `model.decode()` và vẽ biểu đồ Trellis giải thích tương quan số khung hình với trường độ âm vị. |
| **Đánh giá + so sánh DTW** | **1.5** | Baseline DTW chạy trên cùng tập dữ liệu. Bảng so sánh 6 tiêu chí: Accuracy, số mẫu train, thời gian train, thời gian test/trial, confusion pair, khả năng giải thích alignment. |
| **Lexicon + LM extension** | **1.0** | File [lexicon.txt](file:///d:/Lab03_2351260662_LeThiThuyLinh/lexicon.txt) đúng format ASR và có hàm parser thành dict. Xây dựng Bigram LM từ [corpus.txt](file:///d:/Lab03_2351260662_LeThiThuyLinh/corpus.txt) có làm mịn Add-$k$, xuất [bigram_counts.json](file:///d:/Lab03_2351260662_LeThiThuyLinh/bigram_counts.json), thử nghiệm MAP Re-ranking với $\lambda_{LM} \in \{0.0, 0.1, 0.3, 1.0, 3.0\}$. |
| **Thí nghiệm & phân tích lỗi** | **1.0** | Đầy đủ 6 thí nghiệm bắt buộc E1–E6: biến đổi $n\_states \in \{3, 5, 7\}$, so sánh đặc trưng động, phân tích Full Covariance overfit, kiểm tra độ bền vững Mismatch (pitch nữ + noise), đề xuất tiêu chí từ chối Reject Threshold cho từ ngoài từ điển (OOV). |
| **Mã nguồn, báo cáo, tái lập** | **1.0** | Mã nguồn rõ ràng, chạy lại được từ đầu đến cuối không lỗi; hình ảnh, bảng biểu có chú thích đầy đủ; file báo cáo [BaoCao_Lab03.docx](file:///d:/Lab03_2351260662_LeThiThuyLinh/BaoCao_Lab03.docx) chuẩn học thuật 4–6 trang kèm file markdown [BaoCao_Lab03.md](file:///d:/Lab03_2351260662_LeThiThuyLinh/BaoCao_Lab03.md). |
