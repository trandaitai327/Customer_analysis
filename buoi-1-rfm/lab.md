# Lab: RFM Analysis - Thực hành

## Mục tiêu Lab

1. Tạo bảng RFM từ dữ liệu giao dịch
2. Chuẩn hóa feature
3. Phân nhóm khách hàng theo RFM
4. Visualization: heatmap, segment profile

## Bước 1: Chuẩn bị dữ liệu

### 1.1. Tạo dữ liệu mẫu

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
import seaborn as sns

# Tạo dữ liệu giao dịch mẫu
np.random.seed(42)
n_customers = 1000
n_transactions = 5000

# Tạo danh sách khách hàng
customer_ids = [f'CUST_{i:04d}' for i in range(1, n_customers + 1)]

# Tạo dữ liệu giao dịch
transactions = []
start_date = datetime(2023, 1, 1)
end_date = datetime(2024, 12, 1)

for _ in range(n_transactions):
    customer_id = np.random.choice(customer_ids)
    # Tạo ngày giao dịch ngẫu nhiên
    days_between = (end_date - start_date).days
    random_days = np.random.randint(0, days_between)
    transaction_date = start_date + timedelta(days=random_days)
    
    # Tạo giá trị giao dịch (phân bố log-normal)
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

### 1.2. Load dữ liệu từ file (nếu có)

```python
# Nếu có file dữ liệu thực tế
# df_transactions = pd.read_csv('transactions.csv')
# df_transactions['transaction_date'] = pd.to_datetime(df_transactions['transaction_date'])
```

## Bước 2: Tính RFM

### 2.1. Xác định ngày phân tích

```python
# Ngày phân tích (thường là ngày hiện tại hoặc ngày cuối cùng có dữ liệu)
analysis_date = df_transactions['transaction_date'].max()
print(f"Ngày phân tích: {analysis_date}")
```

### 2.2. Tính Recency (R)

```python
# Tính Recency: Số ngày từ lần giao dịch cuối đến ngày phân tích
recency_df = df_transactions.groupby('customer_id')['transaction_date'].max().reset_index()
recency_df['recency'] = (analysis_date - recency_df['transaction_date']).dt.days
recency_df = recency_df[['customer_id', 'recency']]

print(recency_df.head())
print(f"\nThống kê Recency:")
print(recency_df['recency'].describe())
```

### 2.3. Tính Frequency (F)

```python
# Tính Frequency: Số lần giao dịch
frequency_df = df_transactions.groupby('customer_id')['transaction_date'].count().reset_index()
frequency_df.columns = ['customer_id', 'frequency']

print(frequency_df.head())
print(f"\nThống kê Frequency:")
print(frequency_df['frequency'].describe())
```

### 2.4. Tính Monetary (M)

```python
# Tính Monetary: Tổng giá trị giao dịch
monetary_df = df_transactions.groupby('customer_id')['amount'].sum().reset_index()
monetary_df.columns = ['customer_id', 'monetary']

print(monetary_df.head())
print(f"\nThống kê Monetary:")
print(monetary_df['monetary'].describe())
```

### 2.5. Kết hợp RFM

```python
# Merge các bảng RFM
rfm_df = recency_df.merge(frequency_df, on='customer_id')
rfm_df = rfm_df.merge(monetary_df, on='customer_id')

print("Bảng RFM:")
print(rfm_df.head(10))
print(f"\nTổng số khách hàng: {len(rfm_df)}")
```

## Bước 3: Phân nhóm RFM (Quintile)

### 3.1. Tính RFM Score (1-5)

```python
# Tạo hàm tính quintile
def calculate_rfm_score(rfm_df):
    # Recency: Giá trị nhỏ hơn = tốt hơn, nên đảo ngược
    rfm_df['R_score'] = pd.qcut(rfm_df['recency'].rank(method='first'), 
                                 q=5, labels=[5, 4, 3, 2, 1])
    
    # Frequency: Giá trị lớn hơn = tốt hơn
    rfm_df['F_score'] = pd.qcut(rfm_df['frequency'].rank(method='first'), 
                                 q=5, labels=[1, 2, 3, 4, 5])
    
    # Monetary: Giá trị lớn hơn = tốt hơn
    rfm_df['M_score'] = pd.qcut(rfm_df['monetary'].rank(method='first'), 
                                 q=5, labels=[1, 2, 3, 4, 5])
    
    # Chuyển sang int
    rfm_df['R_score'] = rfm_df['R_score'].astype(int)
    rfm_df['F_score'] = rfm_df['F_score'].astype(int)
    rfm_df['M_score'] = rfm_df['M_score'].astype(int)
    
    # Tính RFM Score tổng hợp
    rfm_df['RFM_score'] = (rfm_df['R_score'].astype(str) + 
                           rfm_df['F_score'].astype(str) + 
                           rfm_df['M_score'].astype(str)).astype(int)
    
    return rfm_df

rfm_df = calculate_rfm_score(rfm_df)
print(rfm_df.head(10))
```

### 3.2. Phân nhóm khách hàng

```python
def assign_segment(rfm_df):
    """Phân nhóm khách hàng dựa trên RFM score"""
    def get_segment(row):
        r, f, m = row['R_score'], row['F_score'], row['M_score']
        
        if r >= 4 and f >= 4 and m >= 4:
            return 'Champions'
        elif r >= 3 and f >= 3 and m >= 3:
            return 'Loyal Customers'
        elif r >= 4 and f <= 2:
            return 'New Customers'
        elif r >= 3 and f >= 3 and m <= 2:
            return 'Potential Loyalists'
        elif r <= 2 and f >= 3:
            return 'Need Attention'
        elif r <= 2 and f <= 2 and m >= 3:
            return 'About to Sleep'
        elif r <= 2 and f <= 2 and m <= 2:
            return 'Lost'
        else:
            return 'At Risk'
    
    rfm_df['segment'] = rfm_df.apply(get_segment, axis=1)
    return rfm_df

rfm_df = assign_segment(rfm_df)

# Thống kê phân nhóm
segment_stats = rfm_df.groupby('segment').agg({
    'customer_id': 'count',
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean'
}).round(2)
segment_stats.columns = ['count', 'avg_recency', 'avg_frequency', 'avg_monetary']
segment_stats = segment_stats.sort_values('count', ascending=False)

print("Thống kê theo segment:")
print(segment_stats)
```

## Bước 4: Chuẩn hóa Feature

### 4.1. Min-Max Scaling

```python
from sklearn.preprocessing import MinMaxScaler

# Chuẩn hóa RFM
scaler = MinMaxScaler()
rfm_normalized = scaler.fit_transform(rfm_df[['recency', 'frequency', 'monetary']])
rfm_df_normalized = pd.DataFrame(
    rfm_normalized, 
    columns=['recency_norm', 'frequency_norm', 'monetary_norm'],
    index=rfm_df.index
)

rfm_df = pd.concat([rfm_df, rfm_df_normalized], axis=1)
print(rfm_df[['recency', 'recency_norm', 'frequency', 'frequency_norm', 
              'monetary', 'monetary_norm']].head())
```

### 4.2. Z-score Normalization

```python
from sklearn.preprocessing import StandardScaler

scaler_z = StandardScaler()
rfm_zscore = scaler_z.fit_transform(rfm_df[['recency', 'frequency', 'monetary']])
rfm_df_zscore = pd.DataFrame(
    rfm_zscore,
    columns=['recency_z', 'frequency_z', 'monetary_z'],
    index=rfm_df.index
)

rfm_df = pd.concat([rfm_df, rfm_df_zscore], axis=1)
print(rfm_df[['recency', 'recency_z', 'frequency', 'frequency_z', 
              'monetary', 'monetary_z']].head())
```

## Bước 5: Visualization

### 5.1. RFM Heatmap

```python
# Tạo heatmap R-F với giá trị M trung bình
heatmap_data = rfm_df.pivot_table(
    values='monetary',
    index='R_score',
    columns='F_score',
    aggfunc='mean'
)

plt.figure(figsize=(12, 8))
sns.heatmap(heatmap_data, annot=True, fmt='.0f', cmap='YlOrRd', cbar_kws={'label': 'Avg Monetary'})
plt.title('RFM Heatmap: Average Monetary Value', fontsize=16, fontweight='bold')
plt.xlabel('Frequency Score', fontsize=12)
plt.ylabel('Recency Score', fontsize=12)
plt.tight_layout()
plt.show()
```

### 5.2. Segment Profile

```python
# Bar chart: So sánh R, F, M trung bình giữa các segment
fig, axes = plt.subplots(1, 3, figsize=(18, 5))

segment_avg = rfm_df.groupby('segment').agg({
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean'
}).sort_values('monetary', ascending=False)

# Recency
axes[0].barh(segment_avg.index, segment_avg['recency'], color='skyblue')
axes[0].set_xlabel('Average Recency (days)')
axes[0].set_title('Average Recency by Segment')
axes[0].invert_yaxis()

# Frequency
axes[1].barh(segment_avg.index, segment_avg['frequency'], color='lightgreen')
axes[1].set_xlabel('Average Frequency')
axes[1].set_title('Average Frequency by Segment')
axes[1].invert_yaxis()

# Monetary
axes[2].barh(segment_avg.index, segment_avg['monetary'], color='salmon')
axes[2].set_xlabel('Average Monetary')
axes[2].set_title('Average Monetary by Segment')
axes[2].invert_yaxis()

plt.tight_layout()
plt.show()
```

### 5.3. Phân bố Segment

```python
# Pie chart: Tỷ lệ khách hàng trong mỗi segment
segment_counts = rfm_df['segment'].value_counts()

plt.figure(figsize=(10, 8))
colors = plt.cm.Set3(range(len(segment_counts)))
plt.pie(segment_counts.values, labels=segment_counts.index, autopct='%1.1f%%',
        startangle=90, colors=colors)
plt.title('Customer Distribution by Segment', fontsize=16, fontweight='bold')
plt.axis('equal')
plt.tight_layout()
plt.show()
```

### 5.4. Box Plot: Phân bố giá trị trong mỗi segment

```python
fig, axes = plt.subplots(1, 3, figsize=(18, 6))

# Recency
sns.boxplot(data=rfm_df, x='segment', y='recency', ax=axes[0])
axes[0].set_title('Recency Distribution by Segment')
axes[0].tick_params(axis='x', rotation=45)

# Frequency
sns.boxplot(data=rfm_df, x='segment', y='frequency', ax=axes[1])
axes[1].set_title('Frequency Distribution by Segment')
axes[1].tick_params(axis='x', rotation=45)

# Monetary
sns.boxplot(data=rfm_df, x='segment', y='monetary', ax=axes[2])
axes[2].set_title('Monetary Distribution by Segment')
axes[2].tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.show()
```

### 5.5. Scatter Plot: RFM 3D visualization

```python
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(12, 10))
ax = fig.add_subplot(111, projection='3d')

segments = rfm_df['segment'].unique()
colors = plt.cm.tab10(range(len(segments)))

for i, segment in enumerate(segments):
    segment_data = rfm_df[rfm_df['segment'] == segment]
    ax.scatter(segment_data['recency'], 
               segment_data['frequency'], 
               segment_data['monetary'],
               label=segment, alpha=0.6, s=50, c=[colors[i]])

ax.set_xlabel('Recency')
ax.set_ylabel('Frequency')
ax.set_zlabel('Monetary')
ax.set_title('RFM 3D Visualization by Segment')
ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()
```

## Bước 6: Export kết quả

```python
# Lưu bảng RFM
rfm_df.to_csv('rfm_analysis.csv', index=False)
print("Đã lưu kết quả RFM vào file: rfm_analysis.csv")

# Tạo báo cáo tổng hợp
report = rfm_df.groupby('segment').agg({
    'customer_id': 'count',
    'recency': ['mean', 'std'],
    'frequency': ['mean', 'std'],
    'monetary': ['mean', 'std', 'sum']
}).round(2)

print("\nBáo cáo tổng hợp:")
print(report)
```

## Tổng kết

Sau khi hoàn thành lab, bạn đã:
- ✅ Tạo được bảng RFM từ dữ liệu giao dịch
- ✅ Phân nhóm khách hàng theo RFM score
- ✅ Chuẩn hóa các feature
- ✅ Tạo các visualization: heatmap, segment profile, box plot, scatter plot
