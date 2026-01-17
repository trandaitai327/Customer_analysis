# Bài tập: RFM Analysis - So sánh RFM Score và K-means

## Mục tiêu

Sau khi hoàn thành bài tập, bạn sẽ:
- Áp dụng được 2 phương pháp phân nhóm RFM: **RFM Score** và **K-means Clustering**
- So sánh và đánh giá kết quả giữa 2 phương pháp
- Đưa ra quyết định chọn phương pháp phù hợp với tình huống cụ thể

---

## Bài tập 1: Tính RFM và So sánh 2 Phương pháp

### Yêu cầu

Cho dữ liệu giao dịch của một ngân hàng trong file `transactions_bank.csv`:

| customer_id | transaction_date | amount | product_type |
|-------------|------------------|--------|--------------|
| C001 | 2024-01-15 | 500000 | Savings |
| C001 | 2024-02-20 | 1000000 | Investment |
| C002 | 2024-03-10 | 200000 | Savings |
| ... | ... | ... | ... |

**Nhiệm vụ:**

1. **Tính RFM** cho từng khách hàng với ngày phân tích là 2024-12-01

2. **Phương pháp 1 - RFM Score (Quintile)**:
   - Tính R_score, F_score, M_score (1-5)
   - Phân nhóm khách hàng thành các segment chuẩn (Champions, Loyal Customers, At Risk, ...)
   - Thống kê số lượng khách hàng trong mỗi segment

3. **Phương pháp 2 - K-means Clustering**:
   - Chuẩn hóa dữ liệu RFM (dùng StandardScaler hoặc RobustScaler)
   - Xác định số cluster tối ưu K (dùng Elbow method và Silhouette score)
   - Thực hiện K-means clustering
   - Phân tích đặc điểm từng cluster và đặt tên phù hợp

4. **So sánh 2 phương pháp**:
   - So sánh số lượng segment/cluster
   - So sánh phân bố khách hàng (pie chart cho cả 2)
   - Cross-tabulation: Khách hàng được phân loại như thế nào bởi 2 phương pháp?
   - So sánh thống kê RFM trung bình (R, F, M) giữa các segment/cluster

5. **Visualization**:
   - RFM Heatmap cho RFM Score method
   - Segment/Cluster Profile (bar chart) cho cả 2 phương pháp
   - 3D Scatter Plot so sánh cả 2 phương pháp

6. **Nhận xét và Kết luận**:
   - Ưu nhược điểm của mỗi phương pháp trên dataset này
   - Phương pháp nào phù hợp hơn? Tại sao?
   - Đề xuất phương pháp sử dụng cho business

### Gợi ý

```python
# 1. Đọc dữ liệu và tính RFM
df = pd.read_csv('transactions_bank.csv')
df['transaction_date'] = pd.to_datetime(df['transaction_date'])

# Tính RFM
analysis_date = pd.Timestamp('2024-12-01')
rfm_df = calculate_rfm(df, analysis_date)

# 2. RFM Score Method
rfm_df = calculate_rfm_score(rfm_df)
rfm_df = assign_segment_rfm_score(rfm_df)

# 3. K-means Method
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

scaler = StandardScaler()
rfm_scaled = scaler.fit_transform(rfm_df[['recency', 'frequency', 'monetary']])

# Tìm K tối ưu
k_range = range(3, 10)
silhouette_scores = []
for k in k_range:
    kmeans = KMeans(n_clusters=k, random_state=42)
    labels = kmeans.fit_predict(rfm_scaled)
    silhouette_scores.append(silhouette_score(rfm_scaled, labels))

optimal_k = k_range[np.argmax(silhouette_scores)]
kmeans = KMeans(n_clusters=optimal_k, random_state=42)
rfm_df['cluster'] = kmeans.fit_predict(rfm_scaled)

# 4. So sánh và visualization
# ...
```

---

## Bài tập 2: Phân tích theo Sản phẩm với 2 Phương pháp

### Yêu cầu

Dựa trên dữ liệu từ Bài tập 1, thực hiện:

1. **Tính RFM riêng cho từng loại sản phẩm** (`product_type`):
   - Tính RFM cho mỗi khách hàng theo từng loại sản phẩm (Savings, Investment, Loan, etc.)
   - Ví dụ: Khách hàng C001 có RFM cho Savings và RFM riêng cho Investment

2. **Phân nhóm bằng 2 phương pháp**:
   - **RFM Score**: Phân nhóm khách hàng cho từng loại sản phẩm
   - **K-means**: Tạo một mô hình clustering chung (từ RFM của tất cả sản phẩm)

3. **So sánh hành vi khách hàng**:
   - Khách hàng nào có segment nhất quán giữa các sản phẩm? (ví dụ: Champions cho cả Savings và Investment)
   - Khách hàng nào có segment khác nhau? (ví dụ: Champions cho Savings nhưng At Risk cho Investment)
   - Mối quan hệ giữa đa dạng sản phẩm và segment

4. **Visualization**:
   - Heatmap RFM cho từng loại sản phẩm (RFM Score method)
   - Bar chart so sánh Monetary trung bình theo product_type và segment
   - Scatter plot: So sánh RFM giữa các sản phẩm cho cùng một khách hàng

### Gợi ý

```python
# Tính RFM theo product_type
rfm_by_product = df.groupby(['customer_id', 'product_type']).agg({
    'transaction_date': ['max', 'count'],
    'amount': 'sum'
}).reset_index()

# Flatten column names
rfm_by_product.columns = ['customer_id', 'product_type', 'last_date', 'frequency', 'monetary']

# Tính Recency
rfm_by_product['recency'] = (analysis_date - rfm_by_product['last_date']).dt.days

# Áp dụng 2 phương pháp phân nhóm
# ...
```

---

## Bài tập 3: Phân tích xu hướng RFM theo thời gian

### Yêu cầu

Tính RFM tại các thời điểm khác nhau và so sánh kết quả 2 phương pháp:

1. **Tính RFM tại 3 thời điểm**:
   - Tháng 6/2024
   - Tháng 9/2024
   - Tháng 12/2024

2. **Phân nhóm bằng 2 phương pháp tại mỗi thời điểm**:
   - RFM Score: Tính segment cho mỗi thời điểm
   - K-means: Tạo mô hình riêng cho mỗi thời điểm (hoặc dùng cùng 1 model)

3. **Phân tích chuyển đổi segment**:
   - Khách hàng nào chuyển từ segment tốt sang xấu? (Ví dụ: Champions → At Risk)
   - Khách hàng nào cải thiện? (Ví dụ: Lost → New Customers)
   - So sánh độ ổn định phân nhóm giữa 2 phương pháp:
     - RFM Score: Có bao nhiêu % khách hàng giữ nguyên segment?
     - K-means: Có bao nhiêu % khách hàng ở cùng cluster?

4. **Visualization**:
   - Sankey diagram thể hiện chuyển đổi giữa các segment (cho cả 2 phương pháp)
   - Line chart thể hiện số lượng khách hàng trong mỗi segment/cluster theo thời gian
   - Heatmap: Ma trận chuyển đổi segment giữa các thời điểm

5. **Nhận xét**:
   - Phương pháp nào cho kết quả ổn định hơn theo thời gian?
   - Phương pháp nào phù hợp để theo dõi sự thay đổi hành vi khách hàng?

### Gợi ý

```python
analysis_dates = ['2024-06-01', '2024-09-01', '2024-12-01']

results_over_time = []

for date in analysis_dates:
    analysis_date = pd.Timestamp(date)
    df_filtered = df[df['transaction_date'] <= date]
    
    # Tính RFM
    rfm_df_time = calculate_rfm(df_filtered, analysis_date)
    
    # Phương pháp 1: RFM Score
    rfm_df_time = calculate_rfm_score(rfm_df_time)
    rfm_df_time = assign_segment_rfm_score(rfm_df_time)
    rfm_df_time['method'] = 'RFM Score'
    rfm_df_time['analysis_date'] = date
    
    # Phương pháp 2: K-means
    # ...
    
    results_over_time.append(rfm_df_time)

# Merge tất cả kết quả
all_results = pd.concat(results_over_time, ignore_index=True)

# Phân tích chuyển đổi
# ...
```

---

## Bài tập 4: Case Study - Chiến lược Marketing với 2 Phương pháp

### Tình huống

Bạn là Data Analyst tại một ngân hàng. Ngân hàng có ngân sách marketing 100 triệu VNĐ và muốn chạy 3 chiến dịch:

1. **Chiến dịch Retention** (30 triệu): Giữ chân khách hàng VIP
2. **Chiến dịch Win-back** (40 triệu): Thu hút lại khách hàng đã rời bỏ
3. **Chiến dịch Upsell** (30 triệu): Nâng cấp dịch vụ cho khách hàng hiện tại

### Yêu cầu

1. **Phân nhóm bằng 2 phương pháp**:
   - RFM Score: Phân nhóm khách hàng
   - K-means: Phân nhóm khách hàng (chọn K phù hợp)

2. **Xác định đối tượng cho từng chiến dịch** (cho cả 2 phương pháp):

   **RFM Score Method:**
   - Retention: Segment nào? (ví dụ: Champions, Loyal Customers)
   - Win-back: Segment nào? (ví dụ: At Risk, Lost)
   - Upsell: Segment nào? (ví dụ: Potential Loyalists, New Customers)

   **K-means Method:**
   - Retention: Cluster nào? (phân tích đặc điểm để xác định)
   - Win-back: Cluster nào?
   - Upsell: Cluster nào?

3. **So sánh đối tượng giữa 2 phương pháp**:
   - Có bao nhiêu khách hàng **trùng** giữa 2 phương pháp cho mỗi chiến dịch?
   - Có bao nhiêu khách hàng **khác nhau**?
   - Vẽ Venn diagram hoặc table so sánh

4. **Ước tính số lượng và chi phí**:
   - Giả sử chi phí tiếp cận mỗi khách hàng là 50,000 VNĐ
   - Tính số khách hàng có thể tiếp cận với ngân sách cho **mỗi phương pháp**
   - So sánh: Phương pháp nào cho nhiều khách hàng hơn?

5. **Dự đoán kết quả**:
   - Giả sử tỷ lệ phản hồi (response rate):
     - Retention: 20%
     - Win-back: 10%
     - Upsell: 15%
   - Tính số khách hàng dự kiến phản hồi cho **cả 2 phương pháp**
   - So sánh kết quả

6. **Báo cáo và Đề xuất**:
   - Tạo dashboard tổng hợp so sánh 2 phương pháp
   - Đề xuất phương pháp nào nên sử dụng? Tại sao?
   - Đề xuất phân bổ ngân sách tối ưu

### Gợi ý

```python
# Xác định đối tượng - RFM Score
retention_targets_score = rfm_df[rfm_df['segment'].isin(['Champions', 'Loyal Customers'])]
winback_targets_score = rfm_df[rfm_df['segment'].isin(['At Risk', 'Lost'])]
upsell_targets_score = rfm_df[rfm_df['segment'].isin(['Potential Loyalists', 'New Customers'])]

# Xác định đối tượng - K-means (cần phân tích đặc điểm cluster trước)
# Ví dụ: Cluster có R thấp, F cao, M cao → Retention
# Cluster có R cao, F thấp, M thấp → Win-back

# So sánh
from matplotlib_venn import venn2, venn3

# Venn diagram cho Retention
venn2([set(retention_targets_score['customer_id']), 
       set(retention_targets_kmeans['customer_id'])],
      set_labels=('RFM Score', 'K-means'))
```

---

## Bài tập 5: Nâng cao - Kết hợp 2 Phương pháp

### Yêu cầu

Thay vì chọn 1 trong 2 phương pháp, hãy kết hợp cả 2:

1. **Hybrid Approach - 2 Bước**:
   - **Bước 1**: Dùng **K-means** để phát hiện số lượng cluster tự nhiên (K tối ưu)
   - **Bước 2**: Dùng **RFM Score** logic để đặt tên và giải thích từng cluster

2. **Ensemble Method**:
   - Chạy cả 2 phương pháp độc lập
   - Gán khách hàng vào segment dựa trên **consensus**:
     - Nếu cả 2 phương pháp đều phân khách hàng vào segment tốt → Khách hàng **chắc chắn** tốt
     - Nếu cả 2 đều phân vào segment xấu → Khách hàng **chắc chắn** xấu
     - Nếu kết quả khác nhau → Khách hàng **cần xem xét thêm**

3. **Tạo Segment mới dựa trên kết quả**:
   - **High Confidence Champions**: Khách hàng được cả 2 phương pháp xếp vào Champions/Cluster tốt nhất
   - **Uncertain**: Khách hàng có kết quả khác nhau giữa 2 phương pháp
   - **High Confidence At Risk**: Khách hàng được cả 2 phương pháp xếp vào At Risk/Cluster xấu nhất

4. **So sánh với phương pháp đơn lẻ**:
   - Ensemble method có tốt hơn không?
   - Segment nào có số lượng thay đổi nhiều nhất?

5. **Visualization**:
   - Heatmap thể hiện agreement giữa 2 phương pháp
   - Bar chart so sánh số lượng khách hàng trong mỗi segment (đơn lẻ vs ensemble)

### Gợi ý

```python
# 1. Kết hợp kết quả
rfm_comparison['segment_ensemble'] = rfm_comparison.apply(
    lambda row: create_ensemble_segment(row['segment_score'], row['segment_kmeans']), 
    axis=1
)

def create_ensemble_segment(score_seg, kmeans_seg):
    """Tạo segment dựa trên consensus của 2 phương pháp"""
    # Định nghĩa mapping segment tốt/xấu
    good_segments = ['Champions', 'Loyal Customers']
    bad_segments = ['At Risk', 'Lost']
    
    if score_seg in good_segments and kmeans_seg in good_clusters:
        return 'High Confidence Champions'
    elif score_seg in bad_segments and kmeans_seg in bad_clusters:
        return 'High Confidence At Risk'
    else:
        return 'Uncertain'

# 2. So sánh
# ...
```

---

## Bài tập 6: Thực hành với dữ liệu thực tế

### Yêu cầu

Tìm hoặc tạo dataset về giao dịch ngân hàng/fintech và thực hiện:

1. **Data cleaning**:
   - Xử lý missing values
   - Xử lý outliers (đặc biệt quan trọng cho K-means)
   - Validate dữ liệu

2. **RFM Analysis đầy đủ với 2 phương pháp**:
   - Tính RFM
   - Phân nhóm bằng **RFM Score**
   - Phân nhóm bằng **K-means**
   - So sánh và đánh giá

3. **Insights & Recommendations**:
   - Phát hiện patterns thú vị từ cả 2 phương pháp
   - So sánh: Phương pháp nào cho insights tốt hơn?
   - Đề xuất chiến lược marketing dựa trên kết quả
   - Tạo báo cáo tổng hợp so sánh 2 phương pháp

4. **Presentation**:
   - Tạo dashboard với **Plotly** hoặc **Streamlit** để so sánh 2 phương pháp
   - Có thể switch giữa RFM Score và K-means để xem kết quả
   - Viết báo cáo phân tích (3-5 trang) bao gồm:
     - So sánh 2 phương pháp
     - Kết luận và đề xuất

### Gợi ý nguồn dữ liệu

- **Kaggle**: 
  - Bank Customer Segmentation
  - Online Retail Dataset (cho RFM)
  - E-commerce Customer Segmentation
- **UCI Machine Learning Repository**: Banking datasets
- **Tạo synthetic data** với các đặc điểm thực tế (như trong lab)

### Cấu trúc Dashboard (gợi ý với Streamlit)

```python
import streamlit as st

st.title("RFM Analysis - So sánh 2 Phương pháp")

# Sidebar - Chọn phương pháp
method = st.sidebar.selectbox("Chọn phương pháp", 
                               ["RFM Score", "K-means", "So sánh"])

if method == "RFM Score":
    # Hiển thị kết quả RFM Score
    st.header("RFM Score Method")
    # ... visualization
    
elif method == "K-means":
    # Hiển thị kết quả K-means
    st.header("K-means Method")
    # ... visualization
    
else:
    # So sánh 2 phương pháp
    st.header("So sánh 2 Phương pháp")
    # ... comparison visualization
```

---

## Hướng dẫn nộp bài

1. **Tạo folder riêng cho mỗi bài tập**
2. **Bao gồm**:
   - Code (Python notebook hoặc script)
   - Kết quả (visualizations, tables)
   - Báo cáo ngắn so sánh 2 phương pháp (cho mỗi bài tập)
3. **Comment code rõ ràng**, đặc biệt phần so sánh
4. **Nộp qua Google Drive hoặc GitHub**

---

## Tiêu chí đánh giá

- **Hoàn thành đúng yêu cầu**: 30%
  - Áp dụng đúng cả 2 phương pháp
  - So sánh đầy đủ các khía cạnh
  
- **Code chất lượng**: 25%
  - Code clean, có comment
  - Sử dụng đúng thư viện và hàm
  
- **So sánh và Phân tích**: 25%
  - So sánh sâu giữa 2 phương pháp
  - Nhận xét có ý nghĩa
  
- **Visualization & Presentation**: 20%
  - Visualization rõ ràng, dễ hiểu
  - Dashboard/báo cáo chuyên nghiệp

---

## Tài liệu tham khảo

- [RFM Analysis in Python](https://towardsdatascience.com/rfm-analysis-in-python-65b0e3e1c19a)
- [Customer Segmentation with RFM](https://www.kaggle.com/code/arindam235/startup-customer-segmentation-rfm-analysis)
- [K-means Clustering for Customer Segmentation](https://towardsdatascience.com/customer-segmentation-using-k-means-clustering-d33964f238c2)
- [Comparing Clustering Methods](https://scikit-learn.org/stable/modules/clustering.html#overview-of-clustering-methods)