# Bài tập: RFM Analysis

## Bài tập 1: Tính RFM cơ bản

### Yêu cầu
Cho dữ liệu giao dịch của một ngân hàng trong file `transactions_bank.csv`:

| customer_id | transaction_date | amount | product_type |
|-------------|------------------|--------|--------------|
| C001 | 2024-01-15 | 500000 | Savings |
| C001 | 2024-02-20 | 1000000 | Investment |
| C002 | 2024-03-10 | 200000 | Savings |
| ... | ... | ... | ... |

**Nhiệm vụ:**
1. Tính RFM cho từng khách hàng với ngày phân tích là 2024-12-01
2. Phân nhóm khách hàng thành 5 nhóm theo RFM score
3. Xác định top 10 khách hàng có giá trị cao nhất (Champions)

### Gợi ý
- Sử dụng pandas để đọc và xử lý dữ liệu
- Tính Recency: số ngày từ lần giao dịch cuối đến 2024-12-01
- Tính Frequency: số lần giao dịch
- Tính Monetary: tổng giá trị giao dịch

---

## Bài tập 2: Phân tích theo sản phẩm

### Yêu cầu
Dựa trên dữ liệu từ Bài tập 1, thực hiện:

1. **RFM theo từng loại sản phẩm**: Tính RFM riêng cho từng `product_type` (Savings, Investment, Loan, etc.)

2. **So sánh hành vi khách hàng**:
   - Khách hàng nào chỉ sử dụng 1 loại sản phẩm?
   - Khách hàng nào đa dạng sản phẩm (sử dụng nhiều loại)?
   - Mối quan hệ giữa đa dạng sản phẩm và Monetary value

3. **Visualization**:
   - Heatmap RFM cho từng loại sản phẩm
   - Bar chart so sánh Monetary trung bình theo product_type

### Gợi ý
```python
# Tính RFM theo product_type
rfm_by_product = df.groupby(['customer_id', 'product_type']).agg({
    'transaction_date': ['max', 'count'],
    'amount': 'sum'
})
```

---

## Bài tập 3: Phân tích xu hướng RFM theo thời gian

### Yêu cầu
Tính RFM tại các thời điểm khác nhau và phân tích xu hướng:

1. **Tính RFM tại 3 thời điểm**:
   - Tháng 6/2024
   - Tháng 9/2024
   - Tháng 12/2024

2. **Phân tích chuyển đổi segment**:
   - Khách hàng nào chuyển từ segment tốt sang xấu? (Ví dụ: Champions → At Risk)
   - Khách hàng nào cải thiện? (Ví dụ: Lost → New Customers)
   - Tỷ lệ churn (khách hàng biến mất khỏi dữ liệu)

3. **Visualization**:
   - Sankey diagram thể hiện chuyển đổi giữa các segment
   - Line chart thể hiện số lượng khách hàng trong mỗi segment theo thời gian

### Gợi ý
```python
# Tính RFM tại từng thời điểm
analysis_dates = ['2024-06-01', '2024-09-01', '2024-12-01']

for date in analysis_dates:
    # Filter dữ liệu đến ngày đó
    df_filtered = df[df['transaction_date'] <= date]
    # Tính RFM
    # ...
```

---

## Bài tập 4: Case Study - Chiến lược Marketing

### Tình huống
Bạn là Data Analyst tại một ngân hàng. Ngân hàng có ngân sách marketing 100 triệu VNĐ và muốn chạy 3 chiến dịch:

1. **Chiến dịch Retention** (30 triệu): Giữ chân khách hàng VIP
2. **Chiến dịch Win-back** (40 triệu): Thu hút lại khách hàng đã rời bỏ
3. **Chiến dịch Upsell** (30 triệu): Nâng cấp dịch vụ cho khách hàng hiện tại

### Yêu cầu
Dựa trên phân tích RFM:

1. **Xác định đối tượng cho từng chiến dịch**:
   - Retention: Khách hàng nào?
   - Win-back: Khách hàng nào?
   - Upsell: Khách hàng nào?

2. **Ước tính số lượng khách hàng**:
   - Giả sử chi phí tiếp cận mỗi khách hàng là 50,000 VNĐ
   - Tính số khách hàng có thể tiếp cận với ngân sách

3. **Dự đoán kết quả**:
   - Giả sử tỷ lệ phản hồi (response rate):
     - Retention: 20%
     - Win-back: 10%
     - Upsell: 15%
   - Tính số khách hàng dự kiến phản hồi

4. **Báo cáo**:
   - Tạo dashboard tổng hợp
   - Đề xuất phân bổ ngân sách tối ưu

### Gợi ý
```python
# Xác định đối tượng
retention_targets = rfm_df[rfm_df['segment'].isin(['Champions', 'Loyal Customers'])]
winback_targets = rfm_df[rfm_df['segment'].isin(['At Risk', 'Lost'])]
upsell_targets = rfm_df[rfm_df['segment'].isin(['Potential Loyalists', 'New Customers'])]
```

---

## Bài tập 5: Nâng cao - RFM với Machine Learning

### Yêu cầu
Sử dụng RFM features để dự đoán khả năng churn:

1. **Tạo target variable**:
   - Khách hàng churn nếu không có giao dịch trong 90 ngày cuối
   - Label: 1 = churn, 0 = active

2. **Feature engineering**:
   - Sử dụng R, F, M làm features
   - Thêm các features: R/F ratio, M/F ratio, tenure, etc.

3. **Train model**:
   - Sử dụng Logistic Regression hoặc Random Forest
   - Đánh giá với AUC, Precision, Recall

4. **Phân tích**:
   - Feature importance
   - Xác định ngưỡng RFM nào dễ churn nhất

### Gợi ý
```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import roc_auc_score, classification_report

# Tạo target
rfm_df['churn'] = (rfm_df['recency'] > 90).astype(int)

# Features
features = ['recency', 'frequency', 'monetary', 'R_score', 'F_score', 'M_score']
X = rfm_df[features]
y = rfm_df['churn']

# Train model
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Evaluate
y_pred_proba = model.predict_proba(X_test)[:, 1]
auc = roc_auc_score(y_test, y_pred_proba)
print(f"AUC: {auc}")
```

---

## Bài tập 6: Thực hành với dữ liệu thực tế

### Yêu cầu
Tìm hoặc tạo dataset về giao dịch ngân hàng/fintech và thực hiện:

1. **Data cleaning**:
   - Xử lý missing values
   - Xử lý outliers
   - Validate dữ liệu

2. **RFM Analysis đầy đủ**:
   - Tính RFM
   - Phân nhóm
   - Chuẩn hóa

3. **Insights & Recommendations**:
   - Phát hiện patterns thú vị
   - Đề xuất chiến lược marketing
   - Tạo báo cáo tổng hợp

4. **Presentation**:
   - Tạo dashboard với Plotly hoặc Streamlit
   - Viết báo cáo phân tích (2-3 trang)

### Gợi ý nguồn dữ liệu
- Kaggle: Banking datasets
- UCI Machine Learning Repository
- Tạo synthetic data với các đặc điểm thực tế

---

## Hướng dẫn nộp bài

1. Tạo folder riêng cho mỗi bài tập
2. Bao gồm:
   - Code (Python notebook hoặc script)
   - Kết quả (visualizations, tables)
   - Báo cáo ngắn (nếu có)
3. Comment code rõ ràng
4. Nộp qua Google Drive hoặc GitHub

## Tiêu chí đánh giá

- **Hoàn thành đúng yêu cầu**: 40%
- **Code chất lượng**: 30%
- **Insights & Analysis**: 20%
- **Visualization & Presentation**: 10%

---

## Tài liệu tham khảo

- [RFM Analysis in Python](https://towardsdatascience.com/rfm-analysis-in-python-65b0e3e1c19a)
- [Customer Segmentation with RFM](https://www.kaggle.com/code/arindam235/startup-customer-segmentation-rfm-analysis)
- [RFM Analysis Best Practices](https://www.putler.com/rfm-analysis/)
