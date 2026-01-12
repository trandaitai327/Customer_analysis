# Lab: BG-NBD & Gamma-Gamma → Predictive CLV

## Mục tiêu Lab

1. Huấn luyện BG-NBD & Gamma-Gamma bằng Python
2. Tính CLV cho từng khách hàng
3. Xếp hạng khách hàng theo CLV
4. Phân vùng "High/Medium/Low Future Value" để dùng cho marketing

## Bước 1: Cài đặt thư viện

```python
# Cài đặt lifetimes
# pip install lifetimes

import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
import seaborn as sns
from lifetimes import BetaGeoFitter
from lifetimes import GammaGammaFitter
from lifetimes.utils import calibration_and_holdout_data
from lifetimes.utils import summary_data_from_transaction_data
import warnings
warnings.filterwarnings('ignore')
```

## Bước 2: Chuẩn bị dữ liệu

### 2.1. Tạo dữ liệu mẫu

```python
# Tạo dữ liệu giao dịch mẫu
np.random.seed(42)
n_customers = 1000
n_transactions = 5000

customer_ids = [f'CUST_{i:04d}' for i in range(1, n_customers + 1)]
start_date = datetime(2023, 1, 1)
end_date = datetime(2024, 12, 1)

transactions = []
for _ in range(n_transactions):
    customer_id = np.random.choice(customer_ids)
    days_between = (end_date - start_date).days
    random_days = np.random.randint(0, days_between)
    transaction_date = start_date + timedelta(days=random_days)
    
    # Giá trị giao dịch (phân bố log-normal)
    transaction_amount = np.random.lognormal(mean=5, sigma=1)
    
    transactions.append({
        'customer_id': customer_id,
        'transaction_date': transaction_date,
        'amount': transaction_amount
    })

df_transactions = pd.DataFrame(transactions)
df_transactions['transaction_date'] = pd.to_datetime(df_transactions['transaction_date'])
df_transactions = df_transactions.sort_values('transaction_date')

print(f"Tổng số giao dịch: {len(df_transactions)}")
print(f"Số khách hàng: {df_transactions['customer_id'].nunique()}")
print(df_transactions.head())
```

### 2.2. Chuyển đổi dữ liệu sang format của lifetimes

```python
# lifetimes cần format: customer_id, date, monetary_value
df_lifetimes = df_transactions[['customer_id', 'transaction_date', 'amount']].copy()
df_lifetimes.columns = ['customer_id', 'date', 'monetary_value']

print(df_lifetimes.head())
```

### 2.3. Tạo summary data

```python
# Tạo summary data cho BG-NBD
# x: số lần giao dịch
# recency: khoảng thời gian từ lần đầu đến lần cuối
# T: tổng thời gian quan sát
# frequency: số lần giao dịch (trùng với x)
# monetary_value: giá trị trung bình

summary = summary_data_from_transaction_data(
    df_lifetimes,
    customer_id_col='customer_id',
    datetime_col='date',
    monetary_value_col='monetary_value',
    observation_period_end=end_date
)

print(f"Số khách hàng: {len(summary)}")
print(summary.head(10))
print("\nThống kê:")
print(summary.describe())
```

## Bước 3: Huấn luyện BG-NBD Model

### 3.1. Fit BG-NBD

```python
# Khởi tạo model
bgf = BetaGeoFitter(penalizer_coef=0.0)

# Fit model
bgf.fit(
    summary['frequency'],
    summary['recency'],
    summary['T']
)

# In tham số
print("BG-NBD Parameters:")
print(f"r: {bgf.r_}")
print(f"alpha: {bgf.alpha_}")
print(f"a: {bgf.a_}")
print(f"b: {bgf.b_}")
```

### 3.2. Dự đoán số giao dịch tương lai

```python
# Dự đoán số giao dịch trong 30 ngày tới
t = 30  # days
summary['predicted_purchases_30d'] = bgf.conditional_expected_number_of_purchases_up_to_time(
    t,
    summary['frequency'],
    summary['recency'],
    summary['T']
)

# Dự đoán số giao dịch trong 90 ngày tới
summary['predicted_purchases_90d'] = bgf.conditional_expected_number_of_purchases_up_to_time(
    90,
    summary['frequency'],
    summary['recency'],
    summary['T']
)

print(summary[['frequency', 'predicted_purchases_30d', 'predicted_purchases_90d']].head(10))
```

### 3.3. Tính xác suất khách hàng còn "alive"

```python
# Tính P(alive)
summary['p_alive'] = bgf.conditional_probability_alive(
    summary['frequency'],
    summary['recency'],
    summary['T']
)

print(summary[['frequency', 'recency', 'p_alive']].head(10))
print(f"\nSố khách hàng có P(alive) > 0.5: {(summary['p_alive'] > 0.5).sum()}")
```

### 3.4. Visualization BG-NBD

```python
# Plot frequency vs recency với P(alive)
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# Scatter plot: Frequency vs Recency, màu sắc = P(alive)
scatter = axes[0].scatter(
    summary['recency'],
    summary['frequency'],
    c=summary['p_alive'],
    cmap='RdYlGn',
    alpha=0.6,
    s=50
)
axes[0].set_xlabel('Recency')
axes[0].set_ylabel('Frequency')
axes[0].set_title('Frequency vs Recency (colored by P(alive))')
plt.colorbar(scatter, ax=axes[0], label='P(alive)')

# Histogram P(alive)
axes[1].hist(summary['p_alive'], bins=50, edgecolor='black', alpha=0.7)
axes[1].set_xlabel('P(alive)')
axes[1].set_ylabel('Number of Customers')
axes[1].set_title('Distribution of P(alive)')
axes[1].axvline(0.5, color='red', linestyle='--', label='Threshold 0.5')
axes[1].legend()

plt.tight_layout()
plt.show()
```

## Bước 4: Huấn luyện Gamma-Gamma Model

### 4.1. Filter khách hàng có frequency > 0

```python
# Gamma-Gamma chỉ áp dụng cho khách hàng có ít nhất 1 giao dịch
summary_with_money_value = summary[summary['frequency'] > 0].copy()

print(f"Số khách hàng có giao dịch: {len(summary_with_money_value)}")
```

### 4.2. Fit Gamma-Gamma

```python
# Khởi tạo model
ggf = GammaGammaFitter(penalizer_coef=0.0)

# Fit model
ggf.fit(
    summary_with_money_value['frequency'],
    summary_with_money_value['monetary_value']
)

# In tham số
print("Gamma-Gamma Parameters:")
print(f"p: {ggf.p_}")
print(f"q: {ggf.q_}")
print(f"v: {ggf.v_}")
```

### 4.3. Dự đoán giá trị giao dịch trung bình tương lai

```python
# Dự đoán giá trị giao dịch trung bình
summary_with_money_value['predicted_avg_value'] = ggf.conditional_expected_average_profit(
    summary_with_money_value['frequency'],
    summary_with_money_value['monetary_value']
)

print(summary_with_money_value[['monetary_value', 'predicted_avg_value']].head(10))
print(f"\nCorrelation: {summary_with_money_value['monetary_value'].corr(summary_with_money_value['predicted_avg_value']):.3f}")
```

## Bước 5: Tính CLV

### 5.1. Tính CLV cho 30 ngày

```python
# CLV = E[transactions] × E[average transaction value]
summary_with_money_value['clv_30d'] = (
    summary_with_money_value['predicted_purchases_30d'] * 
    summary_with_money_value['predicted_avg_value']
)

print(summary_with_money_value[['predicted_purchases_30d', 'predicted_avg_value', 'clv_30d']].head(10))
```

### 5.2. Tính CLV cho 90 ngày

```python
# CLV 90 ngày
summary_with_money_value['clv_90d'] = (
    summary_with_money_value['predicted_purchases_90d'] * 
    summary_with_money_value['predicted_avg_value']
)

print(summary_with_money_value[['clv_30d', 'clv_90d']].head(10))
```

### 5.3. Tính CLV với discount rate

```python
# CLV với discount rate (ví dụ: 10% mỗi tháng)
discount_rate = 0.1  # 10% per month
months = [1, 2, 3, 6, 12]  # 1, 2, 3, 6, 12 tháng

for month in months:
    days = month * 30
    # Dự đoán số giao dịch
    predicted_purchases = bgf.conditional_expected_number_of_purchases_up_to_time(
        days,
        summary_with_money_value['frequency'],
        summary_with_money_value['recency'],
        summary_with_money_value['T']
    )
    
    # CLV với discount
    clv_discounted = 0
    for m in range(1, month + 1):
        monthly_purchases = predicted_purchases / month
        discounted_value = (monthly_purchases * 
                          summary_with_money_value['predicted_avg_value']) / ((1 + discount_rate) ** m)
        clv_discounted += discounted_value
    
    summary_with_money_value[f'clv_{month}m_discounted'] = clv_discounted

print(summary_with_money_value[['clv_30d', 'clv_1m_discounted', 'clv_3m_discounted', 'clv_6m_discounted']].head(10))
```

## Bước 6: Xếp hạng khách hàng theo CLV

### 6.1. Xếp hạng

```python
# Xếp hạng theo CLV 90 ngày
summary_with_money_value['clv_rank'] = summary_with_money_value['clv_90d'].rank(ascending=False, method='dense')

# Top 10 khách hàng
top_customers = summary_with_money_value.nlargest(10, 'clv_90d')[
    ['frequency', 'recency', 'monetary_value', 'p_alive', 
     'predicted_purchases_90d', 'predicted_avg_value', 'clv_90d', 'clv_rank']
]

print("Top 10 khách hàng theo CLV:")
print(top_customers)
```

### 6.2. Phân vùng High/Medium/Low Future Value

```python
# Phân vùng theo percentile
clv_75 = summary_with_money_value['clv_90d'].quantile(0.75)
clv_25 = summary_with_money_value['clv_90d'].quantile(0.25)

def assign_clv_segment(clv):
    if clv >= clv_75:
        return 'High'
    elif clv >= clv_25:
        return 'Medium'
    else:
        return 'Low'

summary_with_money_value['clv_segment'] = summary_with_money_value['clv_90d'].apply(assign_clv_segment)

# Thống kê theo segment
segment_stats = summary_with_money_value.groupby('clv_segment').agg({
    'customer_id': 'count',
    'clv_90d': ['mean', 'median', 'sum'],
    'p_alive': 'mean',
    'predicted_purchases_90d': 'mean',
    'predicted_avg_value': 'mean'
}).round(2)

print("Thống kê theo CLV Segment:")
print(segment_stats)
```

## Bước 7: Visualization

### 7.1. CLV Distribution

```python
fig, axes = plt.subplots(2, 2, figsize=(16, 12))

# Histogram CLV
axes[0, 0].hist(summary_with_money_value['clv_90d'], bins=50, edgecolor='black', alpha=0.7)
axes[0, 0].set_xlabel('CLV (90 days)')
axes[0, 0].set_ylabel('Number of Customers')
axes[0, 0].set_title('Distribution of CLV')
axes[0, 0].axvline(clv_75, color='green', linestyle='--', label='75th percentile')
axes[0, 0].axvline(clv_25, color='orange', linestyle='--', label='25th percentile')
axes[0, 0].legend()

# Box plot CLV theo segment
summary_with_money_value.boxplot(column='clv_90d', by='clv_segment', ax=axes[0, 1])
axes[0, 1].set_title('CLV by Segment')
axes[0, 1].set_xlabel('CLV Segment')

# Scatter: P(alive) vs CLV
scatter = axes[1, 0].scatter(
    summary_with_money_value['p_alive'],
    summary_with_money_value['clv_90d'],
    c=summary_with_money_value['frequency'],
    cmap='viridis',
    alpha=0.6,
    s=50
)
axes[1, 0].set_xlabel('P(alive)')
axes[1, 0].set_ylabel('CLV (90 days)')
axes[1, 0].set_title('P(alive) vs CLV (colored by Frequency)')
plt.colorbar(scatter, ax=axes[1, 0], label='Frequency')

# Bar chart: Tổng CLV theo segment
segment_clv_sum = summary_with_money_value.groupby('clv_segment')['clv_90d'].sum()
axes[1, 1].bar(segment_clv_sum.index, segment_clv_sum.values, color=['red', 'orange', 'green'])
axes[1, 1].set_xlabel('CLV Segment')
axes[1, 1].set_ylabel('Total CLV')
axes[1, 1].set_title('Total CLV by Segment')

plt.tight_layout()
plt.show()
```

### 7.2. Customer Segmentation Matrix

```python
# Tạo matrix: P(alive) vs CLV
summary_with_money_value['p_alive_segment'] = pd.cut(
    summary_with_money_value['p_alive'],
    bins=[0, 0.3, 0.7, 1.0],
    labels=['Low', 'Medium', 'High']
)

matrix = pd.crosstab(
    summary_with_money_value['p_alive_segment'],
    summary_with_money_value['clv_segment'],
    values=summary_with_money_value['clv_90d'],
    aggfunc='mean'
)

plt.figure(figsize=(10, 6))
sns.heatmap(matrix, annot=True, fmt='.0f', cmap='YlOrRd', cbar_kws={'label': 'Avg CLV'})
plt.title('Customer Segmentation Matrix: P(alive) vs CLV', fontsize=14, fontweight='bold')
plt.xlabel('CLV Segment')
plt.ylabel('P(alive) Segment')
plt.tight_layout()
plt.show()
```

## Bước 8: Model Validation

### 8.1. Calibration

```python
# Chia dữ liệu: train (2023) và test (2024)
calibration_end = datetime(2024, 1, 1)

summary_cal_holdout = calibration_and_holdout_data(
    df_lifetimes,
    customer_id_col='customer_id',
    datetime_col='date',
    monetary_value_col='monetary_value',
    calibration_period_end=calibration_end,
    observation_period_end=end_date
)

print(f"Số khách hàng trong calibration: {len(summary_cal_holdout)}")
print(summary_cal_holdout.head())
```

### 8.2. Fit trên calibration data

```python
# Fit BG-NBD trên calibration
bgf_cal = BetaGeoFitter(penalizer_coef=0.0)
bgf_cal.fit(
    summary_cal_holdout['frequency_cal'],
    summary_cal_holdout['recency_cal'],
    summary_cal_holdout['T_cal']
)

# Dự đoán trên holdout period
holdout_days = (end_date - calibration_end).days
predicted = bgf_cal.predict(
    holdout_days,
    summary_cal_holdout['frequency_cal'],
    summary_cal_holdout['recency_cal'],
    summary_cal_holdout['T_cal']
)

actual = summary_cal_holdout['frequency_holdout']

# So sánh
comparison = pd.DataFrame({
    'predicted': predicted,
    'actual': actual
})

print("So sánh dự đoán vs thực tế:")
print(comparison.head(10))
print(f"\nMAE: {np.mean(np.abs(predicted - actual)):.2f}")
print(f"RMSE: {np.sqrt(np.mean((predicted - actual)**2)):.2f}")
```

### 8.3. Visualization validation

```python
plt.figure(figsize=(10, 6))
plt.scatter(actual, predicted, alpha=0.5)
plt.plot([0, actual.max()], [0, actual.max()], 'r--', label='Perfect prediction')
plt.xlabel('Actual Frequency')
plt.ylabel('Predicted Frequency')
plt.title('Model Validation: Predicted vs Actual')
plt.legend()
plt.tight_layout()
plt.show()
```

## Bước 9: Export kết quả

```python
# Merge với dữ liệu gốc
final_summary = summary_with_money_value.copy()

# Lưu kết quả
final_summary.to_csv('clv_analysis.csv', index=False)
print("Đã lưu kết quả CLV vào file: clv_analysis.csv")

# Tạo báo cáo
report = final_summary.groupby('clv_segment').agg({
    'customer_id': 'count',
    'clv_90d': ['mean', 'sum'],
    'p_alive': 'mean',
    'predicted_purchases_90d': 'mean',
    'predicted_avg_value': 'mean'
}).round(2)

print("\nBáo cáo tổng hợp:")
print(report)
```

## Tổng kết

Sau khi hoàn thành lab, bạn đã:
- ✅ Huấn luyện BG-NBD model để dự đoán số giao dịch tương lai
- ✅ Huấn luyện Gamma-Gamma model để dự đoán giá trị giao dịch trung bình
- ✅ Tính CLV cho từng khách hàng
- ✅ Xếp hạng và phân vùng khách hàng theo CLV
- ✅ Validate mô hình với calibration/holdout data
