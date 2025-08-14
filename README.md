# Telco Churn Prediction
## 1) Business Problem

Latar belakang. Perusahaan telko kehilangan pelanggan (churn) secara konsisten tiap bulan. Biaya akuisisi pelanggan baru > biaya retensi, sehingga strategi yang rasional adalah mendeteksi dini pelanggan berisiko churn dan melakukan intervensi (promo kontrak, dukungan teknis, bundling layanan).

Siapa pengguna solusi ini:
- Tim Retensi/CX: prioritisasi daftar pelanggan berisiko tinggi untuk outbound/penawaran.
- Marketing: desain promo preventif berbasis segmen/fitur pemicu churn.
- Product/Network Ops: fokus peningkatan kualitas layanan di faktor yang paling berpengaruh.

Tujuan:
- Model klasifikasi yang mengutamakan recall pada kelas churn (minim false negative), diukur dengan F2-score sebagai metrik utama.
- Output operasional: scoring harian/mingguan + daftar faktor risiko yang bisa ditindaklanjuti.

## 2) Data Understanding

###  Sumber & Nama File
`data_telco_customer_churn.csv` — subset fitur layanan & tagihan pelanggan.

---

###  Schema (Kolom yang Digunakan)
- **Numerik**:  
  - `tenure` — lama berlangganan (bulan)  
  - `MonthlyCharges` — biaya bulanan pelanggan  
- **Kategorikal**:  
  - `Dependents`, `OnlineSecurity`, `OnlineBackup`, `InternetService`,  
    `DeviceProtection`, `TechSupport`, `Contract`, `PaperlessBilling`  
- **Target**:  
  - `Churn` — status berhenti berlangganan (`Yes`/`No` → dipetakan ke `1`/`0`)

---

###  Ukuran Data & Kualitas
| Tahap                  | Jumlah Baris | Perubahan         | Persentase |
|------------------------|--------------|-------------------|------------|
| Awal                   | 4,930        | —                 | —          |
| Setelah drop duplicate | 4,853        | −77               | −1.56%     |
| Setelah filter outlier tenure < 1 | 4,845        | −8                | −0.16%     |

- **Missing values**: Tidak ada pada kolom yang dimodelkan.

---

###  Data Cleaning (Diterapkan)
1. **Drop duplicates** → mencegah bias pada proses pelatihan.  
2. **Filter `tenure < 1`** → membuang anomali atau entri tidak valid.  
3. **Normalisasi kategori internet**:  
   - Nilai `"No internet service"` pada kolom `OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport` disetarakan menjadi `"No"` → mengurangi sparsity saat one-hot encoding.

---
## 3) Modeling
Problem type. Supervised classification (biner): Churn (1) vs No Churn (0)

Train/Test Split.
- train_test_split(..., stratify=y, random_state=42) → menjaga proporsi kelas di train/test

Preprocessing (Pipeline)
- Numerik: StandardScaler
- Kategorikal: OneHotEncoder(handle_unknown="ignore")
- Pipeline dibungkus ColumnTransformer dan disatukan dengan estimator di sklearn.Pipeline

Algoritme yang dievaluasi.
- RandomForestClassifier (class_weight="balanced")
- BaggingClassifier dengan DecisionTree (balanced) sebagai base estimator
- Metrik utama: F2-score (menekankan recall churn)
- Validasi: cross-validation + GridSearchCV untuk hyperparameter kunci (mis. max_depth, n_estimators, max_features, min_samples_split)

3.1 Hasil Tanpa Feature Selection (All Features)
| Model              | F2-test   | Accuracy | Churn (1) — Precision | Churn (1) — Recall | Non-churn (0) — Precision | Non-churn (0) — Recall |
| ------------------ | --------- | -------- | --------------------- | ------------------ | ------------------------- | ---------------------- |
| Random Forest      | 0.741     | 0.73     | 0.49                  | 0.84               | 0.92                      | 0.68                   |
| **Bagging (Best)** | **0.756** | 0.74     | 0.50                  | **0.86**           | 0.93                      | 0.69                   |

3.2 Hasil dengan Feature Selection
Fitur yang digunakan:
- tenure
- MonthlyCharges
- Contract
- InternetService


| Model         | F2-test | Accuracy  | Churn (1) — Precision | Churn (1) — Recall |
| ------------- | ------- | --------- | --------------------- | ------------------ |
| Random Forest | 0.745   | 0.74–0.75 | \~0.52                | 0.84               |
| Bagging       | 0.755   | 0.75      | \~0.52                | 0.83               |

## 4) Conclusion & Recommendation
---

###  Conclusion
- **Manual Approach**  
  - Memiliki performa yang tidak bisa diabaikan, terutama jika fokus pada kerugian langsung.  

- **Machine Learning – Random Forest vs Bagging**  
  - Salah satunya unggul di efisiensi biaya, yang lain unggul di konsistensi prediksi.  
  - Stabilitas *cross-validation* menjadi poin penting dalam evaluasi.  

- **Feature Selection**  
  - Membuat model lebih ramping, tapi ada harga yang harus dibayar.  

---

###  Recommendation
1. **Saat Ini**  
   - Gunakan pendekatan yang paling aman untuk menekan risiko kerugian.  

2. **Ke Depan**  
   - Pertimbangkan dengan model ML yang lebih konsisten walaupun tidak selalu menghasilkan “angka terbaik” di awal.  

---



