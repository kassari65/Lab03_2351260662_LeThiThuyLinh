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

1. Mở file trong Visual Studio Code hoặc Jupyter Lab / Jupyter Notebook.
2. Chọn Kernel Python 3 phù hợp (đã cài các thư viện `librosa`, `hmmlearn`, `scikit-learn`, `matplotlib`).
3. Nhấn **"Run All"** để thực thi toàn bộ pipeline từ đầu đến cuối, quan sát trực tiếp các biểu đồ Heatmap phổ, Trellis Viterbi Alignment, ma trận nhầm lẫn Confusion Matrix và các bảng so sánh.
