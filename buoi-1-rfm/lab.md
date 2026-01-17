# Lab: RFM Analysis - So sánh RFM Score và K-means Clustering

## Mục tiêu Lab

1. Tạo bảng RFM từ dữ liệu giao dịch
2. Phân nhóm khách hàng bằng **RFM Score (Quintile)**
3. Phân nhóm khách hàng bằng **K-means Clustering**
4. So sánh kết quả 2 phương pháp
5. Visualization: heatmap, segment profile, cluster comparison

---

## Bước 1: Chuẩn bị dữ liệu

### 1.1. Import thư viện cần thiết

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.preprocessing import StandardScaler, RobustScaler
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score
import warnings
warnings.filterwarnings('ignore')

# Cấu hình hiển thị
plt.style.use('seaborn-v0_8-darkgrid')
sns.set_palette("husl")
```

### 1.2. Tạo dữ liệu mẫu

```python
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
    transaction_amount = np.random.lognormal(mean=5, sigma=1) * 100000  # Nhân với 100k để có giá trị lớn
    
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
print("\nThống kê giá trị giao dịch:")
print(df_transactions['amount'].describe())
print("\nMẫu dữ liệu:")
print(df_transactions.head(10))
```

### 1.3. Load dữ liệu từ file (nếu có)

```python
# Nếu có file dữ liệu thực tế, uncomment và sử dụng:
# df_transactions = pd.read_csv('transactions.csv')
# df_transactions['transaction_date'] = pd.to_datetime(df_transactions['transaction_date'])
```

---

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

print("Mẫu Recency:")
print(recency_df.head())
print(f"\nThống kê Recency:")
print(recency_df['recency'].describe())
```

### 2.3. Tính Frequency (F)

```python
# Tính Frequency: Số lần giao dịch
frequency_df = df_transactions.groupby('customer_id')['transaction_date'].count().reset_index()
frequency_df.columns = ['customer_id', 'frequency']

print("Mẫu Frequency:")
print(frequency_df.head())
print(f"\nThống kê Frequency:")
print(frequency_df['frequency'].describe())
```

### 2.4. Tính Monetary (M)

```python
# Tính Monetary: Tổng giá trị giao dịch
monetary_df = df_transactions.groupby('customer_id')['amount'].sum().reset_index()
monetary_df.columns = ['customer_id', 'monetary']

print("Mẫu Monetary:")
print(monetary_df.head())
print(f"\nThống kê Monetary:")
print(monetary_df['monetary'].describe())
```

### 2.5. Kết hợp bảng RFM

```python
# Merge các bảng RFM
rfm_df = recency_df.merge(frequency_df, on='customer_id')
rfm_df = rfm_df.merge(monetary_df, on='customer_id')

print("Bảng RFM:")
print(rfm_df.head(10))
print(f"\nTổng số khách hàng: {len(rfm_df)}")
print(f"\nThống kê tổng hợp RFM:")
print(rfm_df[['recency', 'frequency', 'monetary']].describe())
```

---

## Bước 3: Phương pháp 1 - RFM Score (Quintile)

### 3.1. Tính RFM Score (1-5)

```python
# Tạo hàm tính quintile
def calculate_rfm_score(rfm_df):
    """Tính RFM Score (1-5) cho từng chỉ số"""
    rfm_df = rfm_df.copy()
    
    # Recency: Giá trị nhỏ hơn = tốt hơn, nên đảo ngược
    rfm_df['R_score'] = pd.qcut(rfm_df['recency'].rank(method='first'), 
                                 q=5, labels=[5, 4, 3, 2, 1], duplicates='drop')
    
    # Frequency: Giá trị lớn hơn = tốt hơn
    rfm_df['F_score'] = pd.qcut(rfm_df['frequency'].rank(method='first'), 
                                 q=5, labels=[1, 2, 3, 4, 5], duplicates='drop')
    
    # Monetary: Giá trị lớn hơn = tốt hơn
    rfm_df['M_score'] = pd.qcut(rfm_df['monetary'].rank(method='first'), 
                                 q=5, labels=[1, 2, 3, 4, 5], duplicates='drop')
    
    # Chuyển sang int
    rfm_df['R_score'] = rfm_df['R_score'].astype(int)
    rfm_df['F_score'] = rfm_df['F_score'].astype(int)
    rfm_df['M_score'] = rfm_df['M_score'].astype(int)
    
    # Tính RFM Score tổng hợp (ví dụ: 555, 444, 111)
    rfm_df['RFM_score'] = (rfm_df['R_score'].astype(str) + 
                           rfm_df['F_score'].astype(str) + 
                           rfm_df['M_score'].astype(str)).astype(int)
    
    return rfm_df

# Tính RFM Score
rfm_df_score = calculate_rfm_score(rfm_df)

print("Bảng RFM với Score:")
print(rfm_df_score[['customer_id', 'recency', 'frequency', 'monetary', 
                    'R_score', 'F_score', 'M_score', 'RFM_score']].head(10))
```

### 3.2. Phân nhóm khách hàng theo RFM Score

```python
def assign_segment_rfm_score(rfm_df):
    """Phân nhóm khách hàng dựa trên RFM Score"""
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
    
    rfm_df['segment_score'] = rfm_df.apply(get_segment, axis=1)
    return rfm_df

# Phân nhóm
rfm_df_score = assign_segment_rfm_score(rfm_df_score)

# Thống kê phân nhóm
segment_stats_score = rfm_df_score.groupby('segment_score').agg({
    'customer_id': 'count',
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean',
    'R_score': 'mean',
    'F_score': 'mean',
    'M_score': 'mean'
}).round(2)
segment_stats_score.columns = ['count', 'avg_recency', 'avg_frequency', 'avg_monetary',
                                'avg_R', 'avg_F', 'avg_M']
segment_stats_score['percentage'] = (segment_stats_score['count'] / len(rfm_df_score) * 100).round(2)
segment_stats_score = segment_stats_score.sort_values('count', ascending=False)

print("\nThống kê phân nhóm theo RFM Score:")
print(segment_stats_score)
```

---

## Bước 4: Phương pháp 2 - K-means Clustering

### 4.1. Chuẩn hóa dữ liệu RFM

**Quan trọng**: K-means cần chuẩn hóa vì R, F, M có đơn vị và scale khác nhau.

```python
# Lưu ý: Với Recency, giá trị nhỏ = tốt hơn
# Có thể đảo ngược Recency trước khi chuẩn hóa (giá trị lớn = tốt hơn)
rfm_for_clustering = rfm_df.copy()
rfm_for_clustering['recency_inverted'] = rfm_for_clustering['recency'].max() - rfm_for_clustering['recency']

# Chuẩn hóa bằng StandardScaler
scaler = StandardScaler()
rfm_scaled = scaler.fit_transform(rfm_for_clustering[['recency_inverted', 'frequency', 'monetary']])

rfm_scaled_df = pd.DataFrame(
    rfm_scaled,
    columns=['recency_norm', 'frequency_norm', 'monetary_norm'],
    index=rfm_for_clustering.index
)

rfm_for_clustering = pd.concat([rfm_for_clustering, rfm_scaled_df], axis=1)

print("Dữ liệu sau khi chuẩn hóa:")
print(rfm_for_clustering[['recency', 'recency_inverted', 'recency_norm', 
                          'frequency', 'frequency_norm', 
                          'monetary', 'monetary_norm']].head())
```

### 4.2. Xác định số cluster tối ưu (K)

#### 4.2.1. Elbow Method

```python
# Tính WCSS (Within-Cluster Sum of Squares) cho các giá trị K từ 2 đến 10
wcss = []
K_range = range(2, 11)

for k in K_range:
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10, max_iter=300)
    kmeans.fit(rfm_scaled)
    wcss.append(kmeans.inertia_)

# Vẽ Elbow curve
plt.figure(figsize=(12, 6))
plt.plot(K_range, wcss, 'bo-', linewidth=2, markersize=8)
plt.xlabel('Number of Clusters (K)', fontsize=12)
plt.ylabel('WCSS (Within-Cluster Sum of Squares)', fontsize=12)
plt.title('Elbow Method for Optimal K', fontsize=14, fontweight='bold')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()

print("WCSS cho từng K:")
for k, wcss_val in zip(K_range, wcss):
    print(f"K={k}: WCSS={wcss_val:.2f}")
```

#### 4.2.2. Silhouette Score

```python
# Tính Silhouette Score cho các giá trị K
silhouette_scores = []

for k in K_range:
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10, max_iter=300)
    cluster_labels = kmeans.fit_predict(rfm_scaled)
    silhouette_avg = silhouette_score(rfm_scaled, cluster_labels)
    silhouette_scores.append(silhouette_avg)

# Vẽ Silhouette scores
plt.figure(figsize=(12, 6))
plt.plot(K_range, silhouette_scores, 'ro-', linewidth=2, markersize=8)
plt.xlabel('Number of Clusters (K)', fontsize=12)
plt.ylabel('Silhouette Score', fontsize=12)
plt.title('Silhouette Score for Optimal K', fontsize=14, fontweight='bold')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()

print("\nSilhouette Score cho từng K:")
for k, score in zip(K_range, silhouette_scores):
    print(f"K={k}: Silhouette Score={score:.3f}")

# Tìm K tối ưu (Silhouette Score cao nhất)
optimal_k = K_range[np.argmax(silhouette_scores)]
print(f"\nK tối ưu (dựa trên Silhouette Score): {optimal_k}")
```

### 4.3. Thực hiện K-means Clustering

```python
# Chọn K (có thể dựa trên Elbow hoặc Silhouette, hoặc yêu cầu business)
# Ở đây chọn K = 5 để dễ so sánh với RFM Score (có ~8 segment)
K = optimal_k  # Hoặc chọn K = 5, 6, 7 tùy ý

kmeans = KMeans(n_clusters=K, random_state=42, n_init=10, max_iter=300)
rfm_for_clustering['cluster_kmeans'] = kmeans.fit_predict(rfm_scaled)

print(f"Đã phân nhóm {len(rfm_for_clustering)} khách hàng thành {K} clusters")
print(f"Silhouette Score: {silhouette_score(rfm_scaled, rfm_for_clustering['cluster_kmeans']):.3f}")
```

### 4.4. Phân tích và đặt tên cluster

```python
# Phân tích đặc điểm từng cluster
cluster_stats = rfm_for_clustering.groupby('cluster_kmeans').agg({
    'customer_id': 'count',
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean'
}).round(2)
cluster_stats.columns = ['count', 'avg_recency', 'avg_frequency', 'avg_monetary']
cluster_stats['percentage'] = (cluster_stats['count'] / len(rfm_for_clustering) * 100).round(2)

# Sắp xếp theo Monetary (giá trị)
cluster_stats = cluster_stats.sort_values('avg_monetary', ascending=False)

print("\nThống kê từng Cluster (K-means):")
print(cluster_stats)

# So sánh với trung bình tổng thể để đặt tên
overall_mean = rfm_for_clustering[['recency', 'frequency', 'monetary']].mean()

def assign_cluster_name(row):
    """Đặt tên cluster dựa trên đặc điểm so với trung bình"""
    r, f, m = row['avg_recency'], row['avg_frequency'], row['avg_monetary']
    
    # Recency càng nhỏ = tốt hơn
    if r < overall_mean['recency'] and f > overall_mean['frequency'] and m > overall_mean['monetary']:
        return 'Champions'
    elif r > overall_mean['recency'] and f < overall_mean['frequency'] and m < overall_mean['monetary']:
        return 'At Risk / Lost'
    elif r < overall_mean['recency'] and f < overall_mean['frequency']:
        return 'New Customers'
    elif r < overall_mean['recency'] and f > overall_mean['frequency'] and m < overall_mean['monetary']:
        return 'Potential Loyalists'
    elif r > overall_mean['recency'] and f > overall_mean['frequency']:
        return 'Need Attention'
    else:
        return f'Cluster {row.name}'

# Gán tên cho cluster
cluster_names = {}
for idx, row in cluster_stats.iterrows():
    cluster_names[idx] = assign_cluster_name(row)

rfm_for_clustering['segment_kmeans'] = rfm_for_clustering['cluster_kmeans'].map(cluster_names)

print("\nTên các cluster:")
for cluster_id, name in cluster_names.items():
    count = len(rfm_for_clustering[rfm_for_clustering['cluster_kmeans'] == cluster_id])
    print(f"Cluster {cluster_id}: {name} ({count} khách hàng)")
```

---

## Bước 5: So sánh 2 Phương pháp

### 5.1. Merge kết quả 2 phương pháp

```python
# Merge 2 bảng kết quả
rfm_comparison = rfm_df_score[['customer_id', 'recency', 'frequency', 'monetary', 
                                'R_score', 'F_score', 'M_score', 'segment_score']].copy()
rfm_comparison = rfm_comparison.merge(
    rfm_for_clustering[['customer_id', 'cluster_kmeans', 'segment_kmeans']], 
    on='customer_id'
)

print("Bảng so sánh 2 phương pháp:")
print(rfm_comparison.head(10))
```

### 5.2. So sánh số lượng segment/cluster

```python
print("\n=== SO SÁNH SỐ LƯỢNG PHÂN NHÓM ===")
print(f"\nRFM Score: {rfm_comparison['segment_score'].nunique()} segment")
print(f"K-means: {rfm_comparison['segment_kmeans'].nunique()} cluster")
```

### 5.3. So sánh phân bố khách hàng

```python
# Phân bố theo RFM Score
dist_score = rfm_comparison['segment_score'].value_counts().sort_values(ascending=False)
dist_score_pct = (dist_score / len(rfm_comparison) * 100).round(2)

# Phân bố theo K-means
dist_kmeans = rfm_comparison['segment_kmeans'].value_counts().sort_values(ascending=False)
dist_kmeans_pct = (dist_kmeans / len(rfm_comparison) * 100).round(2)

print("\n=== PHÂN BỐ KHÁCH HÀNG ===")
print("\nRFM Score Method:")
print(pd.DataFrame({
    'Count': dist_score,
    'Percentage': dist_score_pct
}))

print("\nK-means Method:")
print(pd.DataFrame({
    'Count': dist_kmeans,
    'Percentage': dist_kmeans_pct
}))
```

### 5.4. Cross-tabulation: Khách hàng được phân loại như thế nào bởi 2 phương pháp?

```python
# Cross-tabulation
crosstab = pd.crosstab(rfm_comparison['segment_score'], 
                       rfm_comparison['segment_kmeans'], 
                       margins=True)

print("\n=== CROSS-TABULATION ===")
print("Hàng: RFM Score | Cột: K-means")
print(crosstab)
```

### 5.5. So sánh thống kê RFM giữa 2 phương pháp

```python
# Tính thống kê RFM trung bình cho từng segment/cluster
stats_comparison = []

# RFM Score
stats_score = rfm_comparison.groupby('segment_score').agg({
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean',
    'customer_id': 'count'
}).round(2)
stats_score['method'] = 'RFM Score'

# K-means
stats_kmeans = rfm_comparison.groupby('segment_kmeans').agg({
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean',
    'customer_id': 'count'
}).round(2)
stats_kmeans['method'] = 'K-means'

# Combine
stats_comparison = pd.concat([
    stats_score.rename_axis('segment').reset_index(),
    stats_kmeans.rename_axis('segment').reset_index()
], ignore_index=True)

print("\n=== SO SÁNH THỐNG KÊ RFM ===")
print(stats_comparison)
```

---

## Bước 6: Visualization

### 6.1. RFM Heatmap - RFM Score Method

```python
# RFM Heatmap cho RFM Score
heatmap_data_score = rfm_df_score.pivot_table(
    values='monetary',
    index='R_score',
    columns='F_score',
    aggfunc='mean'
)

plt.figure(figsize=(12, 8))
sns.heatmap(heatmap_data_score, annot=True, fmt='.0f', cmap='YlOrRd', 
            cbar_kws={'label': 'Avg Monetary'})
plt.title('RFM Heatmap (RFM Score Method): Average Monetary Value', 
          fontsize=16, fontweight='bold')
plt.xlabel('Frequency Score', fontsize=12)
plt.ylabel('Recency Score', fontsize=12)
plt.tight_layout()
plt.show()
```

### 6.2. Segment Profile - RFM Score

```python
# Bar chart: So sánh R, F, M trung bình giữa các segment (RFM Score)
fig, axes = plt.subplots(1, 3, figsize=(18, 6))

segment_avg_score = rfm_df_score.groupby('segment_score').agg({
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean'
}).sort_values('monetary', ascending=False)

# Recency
axes[0].barh(segment_avg_score.index, segment_avg_score['recency'], color='skyblue')
axes[0].set_xlabel('Average Recency (days)', fontsize=11)
axes[0].set_title('Average Recency by Segment (RFM Score)', fontsize=12, fontweight='bold')
axes[0].invert_yaxis()

# Frequency
axes[1].barh(segment_avg_score.index, segment_avg_score['frequency'], color='lightgreen')
axes[1].set_xlabel('Average Frequency', fontsize=11)
axes[1].set_title('Average Frequency by Segment (RFM Score)', fontsize=12, fontweight='bold')
axes[1].invert_yaxis()

# Monetary
axes[2].barh(segment_avg_score.index, segment_avg_score['monetary'], color='salmon')
axes[2].set_xlabel('Average Monetary', fontsize=11)
axes[2].set_title('Average Monetary by Segment (RFM Score)', fontsize=12, fontweight='bold')
axes[2].invert_yaxis()

plt.tight_layout()
plt.show()
```

### 6.3. Cluster Profile - K-means

```python
# Bar chart: So sánh R, F, M trung bình giữa các cluster (K-means)
fig, axes = plt.subplots(1, 3, figsize=(18, 6))

cluster_avg_kmeans = rfm_for_clustering.groupby('segment_kmeans').agg({
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean'
}).sort_values('monetary', ascending=False)

# Recency
axes[0].barh(cluster_avg_kmeans.index, cluster_avg_kmeans['recency'], color='coral')
axes[0].set_xlabel('Average Recency (days)', fontsize=11)
axes[0].set_title('Average Recency by Cluster (K-means)', fontsize=12, fontweight='bold')
axes[0].invert_yaxis()

# Frequency
axes[1].barh(cluster_avg_kmeans.index, cluster_avg_kmeans['frequency'], color='lightblue')
axes[1].set_xlabel('Average Frequency', fontsize=11)
axes[1].set_title('Average Frequency by Cluster (K-means)', fontsize=12, fontweight='bold')
axes[1].invert_yaxis()

# Monetary
axes[2].barh(cluster_avg_kmeans.index, cluster_avg_kmeans['monetary'], color='gold')
axes[2].set_xlabel('Average Monetary', fontsize=11)
axes[2].set_title('Average Monetary by Cluster (K-means)', fontsize=12, fontweight='bold')
axes[2].invert_yaxis()

plt.tight_layout()
plt.show()
```

### 6.4. Phân bố Segment/Cluster

```python
# Pie chart: So sánh phân bố giữa 2 phương pháp
fig, axes = plt.subplots(1, 2, figsize=(18, 8))

# RFM Score
segment_counts_score = rfm_df_score['segment_score'].value_counts()
colors1 = plt.cm.Set3(range(len(segment_counts_score)))
axes[0].pie(segment_counts_score.values, labels=segment_counts_score.index, 
            autopct='%1.1f%%', startangle=90, colors=colors1)
axes[0].set_title('Customer Distribution - RFM Score Method', 
                  fontsize=14, fontweight='bold')

# K-means
cluster_counts_kmeans = rfm_for_clustering['segment_kmeans'].value_counts()
colors2 = plt.cm.Pastel1(range(len(cluster_counts_kmeans)))
axes[1].pie(cluster_counts_kmeans.values, labels=cluster_counts_kmeans.index, 
            autopct='%1.1f%%', startangle=90, colors=colors2)
axes[1].set_title('Customer Distribution - K-means Method', 
                  fontsize=14, fontweight='bold')

plt.tight_layout()
plt.show()
```

### 6.5. 3D Scatter Plot - So sánh 2 phương pháp

```python
from mpl_toolkits.mplot3d import Axes3D

# Tạo 2 subplot cạnh nhau
fig = plt.figure(figsize=(20, 8))

# RFM Score
ax1 = fig.add_subplot(121, projection='3d')
segments_score = rfm_df_score['segment_score'].unique()
colors_score = plt.cm.tab10(range(len(segments_score)))

for i, segment in enumerate(segments_score):
    segment_data = rfm_df_score[rfm_df_score['segment_score'] == segment]
    ax1.scatter(segment_data['recency'], 
               segment_data['frequency'], 
               segment_data['monetary'],
               label=segment, alpha=0.6, s=30, c=[colors_score[i]])

ax1.set_xlabel('Recency', fontsize=10)
ax1.set_ylabel('Frequency', fontsize=10)
ax1.set_zlabel('Monetary', fontsize=10)
ax1.set_title('RFM 3D Visualization - RFM Score Method', fontsize=12, fontweight='bold')
ax1.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize=8)

# K-means
ax2 = fig.add_subplot(122, projection='3d')
clusters_kmeans = rfm_for_clustering['cluster_kmeans'].unique()
colors_kmeans = plt.cm.tab20(range(len(clusters_kmeans)))

for i, cluster in enumerate(clusters_kmeans):
    cluster_data = rfm_for_clustering[rfm_for_clustering['cluster_kmeans'] == cluster]
    ax2.scatter(cluster_data['recency'], 
               cluster_data['frequency'], 
               cluster_data['monetary'],
               label=f'Cluster {cluster}', alpha=0.6, s=30, c=[colors_kmeans[i]])

ax2.set_xlabel('Recency', fontsize=10)
ax2.set_ylabel('Frequency', fontsize=10)
ax2.set_zlabel('Monetary', fontsize=10)
ax2.set_title('RFM 3D Visualization - K-means Method', fontsize=12, fontweight='bold')
ax2.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize=8)

plt.tight_layout()
plt.show()
```

---

## Bước 7: Đánh giá và Kết luận

### 7.1. So sánh Ưu - Nhược điểm trong thực tế

```python
print("\n=== ĐÁNH GIÁ 2 PHƯƠNG PHÁP ===\n")

print("RFM Score Method:")
print("✅ Ưu điểm:")
print("  - Dễ hiểu và giải thích")
print("  - Có tiêu chuẩn ngành (Champions, At Risk, ...)")
print("  - Nhanh và không cần chuẩn hóa")
print("  - Kết quả nhất quán (deterministic)")
print("❌ Nhược điểm:")
print("  - Phân nhóm cứng nhắc (chỉ 8-9 segment)")
print("  - Chỉ dựa trên ranking, không tận dụng hết thông tin")

print("\nK-means Method:")
print("✅ Ưu điểm:")
print("  - Tự động tìm patterns phức tạp")
print("  - Linh hoạt số lượng cluster")
print("  - Tận dụng toàn bộ giá trị thực tế")
print("❌ Nhược điểm:")
print("  - Cần chuẩn hóa dữ liệu")
print("  - Khó giải thích (cần phân tích đặc điểm)")
print("  - Kết quả có thể khác nhau mỗi lần chạy")

print("\n=== KẾT LUẬN ===")
print(f"- RFM Score tạo {rfm_comparison['segment_score'].nunique()} segment")
print(f"- K-means tạo {rfm_comparison['segment_kmeans'].nunique()} cluster")
print(f"- Silhouette Score (K-means): {silhouette_score(rfm_scaled, rfm_for_clustering['cluster_kmeans']):.3f}")
```

### 7.2. Lưu kết quả

```python
# Lưu bảng RFM với 2 phương pháp
rfm_comparison.to_csv('rfm_analysis_comparison.csv', index=False)
print("\nĐã lưu kết quả vào: rfm_analysis_comparison.csv")

# Tạo báo cáo tổng hợp
report_score = rfm_df_score.groupby('segment_score').agg({
    'customer_id': 'count',
    'recency': ['mean', 'std'],
    'frequency': ['mean', 'std'],
    'monetary': ['mean', 'std', 'sum']
}).round(2)

report_kmeans = rfm_for_clustering.groupby('segment_kmeans').agg({
    'customer_id': 'count',
    'recency': ['mean', 'std'],
    'frequency': ['mean', 'std'],
    'monetary': ['mean', 'std', 'sum']
}).round(2)

print("\nBáo cáo RFM Score Method:")
print(report_score)

print("\nBáo cáo K-means Method:")
print(report_kmeans)
```

---

## Tổng kết

Sau khi hoàn thành lab, bạn đã:

✅ **Tạo được bảng RFM** từ dữ liệu giao dịch

✅ **Phân nhóm bằng RFM Score (Quintile)** - Phương pháp truyền thống

✅ **Phân nhóm bằng K-means Clustering** - Phương pháp Machine Learning

✅ **So sánh kết quả 2 phương pháp** - Phân bố, thống kê, cross-tabulation

✅ **Visualization** - Heatmap, segment profile, 3D scatter plot cho cả 2 phương pháp

✅ **Hiểu được ưu nhược điểm** của từng phương pháp trong thực tế

**Khuyến nghị sử dụng**:
- **RFM Score**: Dùng cho reporting, quick wins, non-technical team
- **K-means**: Dùng cho deep analysis, research, khi cần flexibility