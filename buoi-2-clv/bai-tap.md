# Bài tập: BG-NBD & Gamma-Gamma → Predictive CLV

## Bài tập 1: Tính CLV cơ bản

### Yêu cầu
Cho dữ liệu giao dịch của một fintech trong file `transactions_fintech.csv`:

| customer_id | transaction_date | amount |
|-------------|------------------|--------|
| C001 | 2024-01-15 | 500000 |
| C001 | 2024-02-20 | 1000000 |
| C002 | 2024-03-10 | 200000 |
| ... | ... | ... |

**Nhiệm vụ:**
1. Tính CLV cho 30, 90, 180 ngày cho từng khách hàng
2. Xác định top 20 khách hàng có CLV cao nhất
3. Phân tích đặc điểm của top 20 khách hàng (frequency, recency, monetary, P(alive))

### Gợi ý
- Sử dụng thư viện `lifetimes`
- Fit BG-NBD và Gamma-Gamma
- Tính CLV với và không có discount rate

---

## Bài tập 2: So sánh Historical Value vs Predictive CLV

### Yêu cầu
1. **Tính Historical Value**:
   - Tổng giá trị giao dịch trong quá khứ
   - Số giao dịch trung bình/tháng

2. **Tính Predictive CLV**:
   - CLV 90 ngày
   - CLV 180 ngày

3. **So sánh**:
   - Khách hàng nào có Historical Value cao nhưng CLV thấp? (có thể đã churn)
   - Khách hàng nào có Historical Value thấp nhưng CLV cao? (tiềm năng)
   - Correlation giữa Historical Value và CLV

4. **Visualization**:
   - Scatter plot: Historical Value vs CLV
   - Phân nhóm khách hàng theo 2 chiều

### Gợi ý
```python
# Tính Historical Value
historical_value = df_transactions.groupby('customer_id')['amount'].sum()

# So sánh
comparison = pd.DataFrame({
    'historical_value': historical_value,
    'clv_90d': clv_90d
})
```

---

## Bài tập 3: CLV với Discount Rate

### Yêu cầu
Tính CLV với các discount rate khác nhau:

1. **Discount rates**: 0%, 5%, 10%, 15%, 20% mỗi tháng
2. **Time horizons**: 1, 3, 6, 12 tháng
3. **Phân tích**:
   - Ảnh hưởng của discount rate đến CLV
   - Ảnh hưởng của time horizon đến CLV
   - Xác định discount rate phù hợp cho business

4. **Visualization**:
   - Heatmap: CLV theo discount rate và time horizon
   - Line chart: CLV theo time horizon với các discount rate khác nhau

### Gợi ý
```python
discount_rates = [0, 0.05, 0.10, 0.15, 0.20]
time_horizons = [1, 3, 6, 12]  # months

for rate in discount_rates:
    for horizon in time_horizons:
        # Tính CLV với discount
        # ...
```

---

## Bài tập 4: Model Validation & Calibration

### Yêu cầu
1. **Chia dữ liệu**:
   - Calibration period: 6 tháng đầu
   - Holdout period: 3 tháng cuối

2. **Fit model trên calibration data**

3. **Dự đoán trên holdout period**

4. **Đánh giá**:
   - MAE (Mean Absolute Error)
   - RMSE (Root Mean Squared Error)
   - MAPE (Mean Absolute Percentage Error)
   - Correlation giữa predicted và actual

5. **Visualization**:
   - Scatter plot: Predicted vs Actual
   - Residual plot
   - Distribution of errors

### Gợi ý
```python
from lifetimes.utils import calibration_and_holdout_data

# Chia dữ liệu
summary_cal_holdout = calibration_and_holdout_data(...)

# Fit và predict
# ...

# Đánh giá
mae = np.mean(np.abs(predicted - actual))
rmse = np.sqrt(np.mean((predicted - actual)**2))
mape = np.mean(np.abs((predicted - actual) / (actual + 1))) * 100
```

---

## Bài tập 5: Case Study - Marketing Strategy với CLV

### Tình huống
Bạn là Data Scientist tại một ngân hàng. Ngân hàng muốn chạy 3 chiến dịch:

1. **VIP Program** (50 triệu): Dành cho khách hàng có CLV cao nhất
2. **Retention Campaign** (30 triệu): Giữ chân khách hàng có CLV cao nhưng P(alive) thấp
3. **Growth Campaign** (20 triệu): Phát triển khách hàng có CLV trung bình

### Yêu cầu
1. **Xác định đối tượng**:
   - VIP Program: Top X khách hàng theo CLV
   - Retention: CLV cao + P(alive) < 0.5
   - Growth: CLV trung bình + P(alive) > 0.7

2. **Tính toán**:
   - Số lượng khách hàng trong mỗi chiến dịch
   - Tổng CLV của mỗi nhóm
   - Chi phí trên mỗi khách hàng
   - ROI dự kiến (giả sử conversion rate)

3. **Đề xuất**:
   - Phân bổ ngân sách tối ưu
   - Chiến lược cho từng nhóm
   - KPI để đo lường hiệu quả

4. **Báo cáo**:
   - Dashboard tổng hợp
   - Báo cáo phân tích (2-3 trang)

### Gợi ý
```python
# Xác định đối tượng
vip_customers = summary.nlargest(100, 'clv_90d')
retention_customers = summary[(summary['clv_90d'] > clv_75) & (summary['p_alive'] < 0.5)]
growth_customers = summary[(summary['clv_90d'].between(clv_25, clv_75)) & (summary['p_alive'] > 0.7)]

# Tính toán
# ...
```

---

## Bài tập 6: CLV Segmentation & Action Plan

### Yêu cầu
1. **Tạo segmentation matrix**:
   - Trục X: CLV (High/Medium/Low)
   - Trục Y: P(alive) (High/Medium/Low)
   - 9 nhóm khách hàng

2. **Phân tích từng nhóm**:
   - Đặc điểm (frequency, recency, monetary)
   - Số lượng khách hàng
   - Tổng CLV

3. **Action plan cho từng nhóm**:
   - Chiến lược marketing
   - Mức độ ưu tiên
   - Ngân sách đề xuất

4. **Visualization**:
   - Heatmap segmentation matrix
   - Bar chart: Số lượng và tổng CLV theo nhóm

### Gợi ý
```python
# Tạo segmentation
summary['clv_segment'] = pd.cut(summary['clv_90d'], bins=3, labels=['Low', 'Medium', 'High'])
summary['p_alive_segment'] = pd.cut(summary['p_alive'], bins=3, labels=['Low', 'Medium', 'High'])

# Matrix
matrix = pd.crosstab(summary['p_alive_segment'], summary['clv_segment'])
```

---

## Bài tập 7: CLV với Product Mix

### Yêu cầu
Dữ liệu có thêm cột `product_type`:

1. **Tính CLV theo từng loại sản phẩm**:
   - CLV cho Savings
   - CLV cho Investment
   - CLV cho Loan
   - etc.

2. **Phân tích**:
   - Khách hàng nào sử dụng nhiều loại sản phẩm?
   - Mối quan hệ giữa product diversity và CLV
   - Sản phẩm nào mang lại CLV cao nhất?

3. **Cross-sell opportunity**:
   - Khách hàng có CLV cao nhưng chỉ dùng 1 sản phẩm
   - Đề xuất sản phẩm để cross-sell

4. **Visualization**:
   - CLV theo product type
   - Product mix của top customers

### Gợi ý
```python
# Tính CLV theo product
for product in df['product_type'].unique():
    product_transactions = df[df['product_type'] == product]
    # Tính CLV cho product này
    # ...
```

---

## Bài tập 8: Advanced - Custom CLV Model

### Yêu cầu
Thay vì dùng lifetimes, tự implement:

1. **Implement BG-NBD từ đầu**:
   - Maximum Likelihood Estimation
   - Tính P(alive)
   - Dự đoán số giao dịch

2. **Implement Gamma-Gamma từ đầu**:
   - MLE
   - Dự đoán giá trị trung bình

3. **So sánh với lifetimes**:
   - Tham số
   - Dự đoán
   - Performance

### Gợi ý
```python
from scipy.optimize import minimize
from scipy.special import gammaln

def bg_nbd_log_likelihood(params, frequency, recency, T):
    r, alpha, a, b = params
    # Implement log-likelihood
    # ...
    return -log_likelihood  # Negative for minimization

# Optimize
result = minimize(bg_nbd_log_likelihood, x0=[1, 1, 1, 1], 
                 args=(frequency, recency, T), method='L-BFGS-B')
```

---

## Hướng dẫn nộp bài

1. Tạo folder riêng cho mỗi bài tập
2. Bao gồm:
   - Code (Python notebook)
   - Kết quả (visualizations, tables)
   - Báo cáo phân tích (nếu có)
3. Comment code rõ ràng
4. Nộp qua Google Drive hoặc GitHub

## Tiêu chí đánh giá

- **Hoàn thành đúng yêu cầu**: 40%
- **Code chất lượng**: 25%
- **Model validation**: 15%
- **Insights & Analysis**: 15%
- **Visualization & Presentation**: 5%

---

## Tài liệu tham khảo

- [Lifetimes Documentation](https://lifetimes.readthedocs.io/)
- [BG-NBD Paper](https://www.researchgate.net/publication/247836660)
- [Customer Lifetime Value Tutorial](https://towardsdatascience.com/predicting-customer-lifetime-value-with-python-1c0c0c92048a)
