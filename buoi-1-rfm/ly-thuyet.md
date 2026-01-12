# Lý thuyết: Customer Value Foundations - RFM & Customer Behavior Insights

## 📑 Mục lục

1. [Tổng quan chương trình & Hành trình khách hàng](#1-tổng-quan-chương-trình--hành-trình-khách-hàng)
   - 1.1. Customer Journey trong Fintech/Banking
   - 1.2. Tại sao Customer Value quan trọng?

2. [Giới thiệu Customer Value và ứng dụng trong Fintech/Banking](#2-giới-thiệu-customer-value-và-ứng-dụng-trong-fintechbanking)
   - 2.1. Customer Value là gì?
   - 2.2. Ứng dụng trong Fintech/Banking

3. [RFM Analysis](#3-rfm-analysis)
   - 3.1. RFM là gì?
   - 3.2. Logic hành vi của RFM
   - 3.3. Cách tính RFM
   - 3.4. Phân nhóm RFM
   - 3.5. Ý nghĩa của RFM trong phân nhóm khách hàng

4. [Phân tích khách hàng theo hành vi lịch sử (Historical Value)](#4-phân-tích-khách-hàng-theo-hành-vi-lịch-sử-historical-value)
   - 4.1. Historical Value Metrics
   - 4.2. Time-based Analysis
   - 4.3. Cohort Analysis

5. [Chuẩn hóa Feature](#5-chuẩn-hóa-feature)
   - 5.1. Tại sao cần chuẩn hóa?
   - 5.2. Phương pháp chuẩn hóa
   - 5.3. Chuẩn hóa RFM

6. [Visualization](#6-visualization)
   - 6.1. RFM Heatmap
   - 6.2. Segment Profile
   - 6.3. Customer Journey Visualization

7. [Best Practices](#7-best-practices)

---

## 1. Tổng quan chương trình & Hành trình khách hàng

### 1.1. Customer Journey trong Fintech/Banking

Hành trình khách hàng trong lĩnh vực fintech/banking thường trải qua các giai đoạn:

1. **Awareness**: Khách hàng biết đến sản phẩm/dịch vụ
2. **Consideration**: Đánh giá và so sánh các lựa chọn
3. **Acquisition**: Đăng ký và sử dụng dịch vụ lần đầu
4. **Onboarding**: Hoàn thiện hồ sơ, xác thực, kích hoạt tài khoản
5. **Activation**: Thực hiện giao dịch đầu tiên
6. **Engagement**: Sử dụng thường xuyên các tính năng
7. **Retention**: Duy trì sử dụng dịch vụ
8. **Advocacy**: Giới thiệu cho người khác

### 1.2. Tại sao Customer Value quan trọng?

- **Tối ưu ngân sách marketing**: Tập trung vào khách hàng có giá trị cao
- **Cá nhân hóa dịch vụ**: Cung cấp trải nghiệm phù hợp với từng nhóm
- **Giảm churn**: Xác định và giữ chân khách hàng quan trọng
- **Tăng revenue**: Upsell/cross-sell đúng đối tượng
- **Quản lý rủi ro**: Đánh giá khả năng trả nợ, rủi ro tín dụng

## 2. Giới thiệu Customer Value và ứng dụng trong Fintech/Banking

### 2.1. Customer Value là gì?

Customer Value là giá trị mà khách hàng mang lại cho doanh nghiệp, có thể đo lường qua:

- **Historical Value**: Giá trị trong quá khứ (tổng số tiền đã chi tiêu, số giao dịch)
- **Current Value**: Giá trị hiện tại (số dư tài khoản, hạn mức tín dụng đang sử dụng)
- **Future Value (CLV)**: Giá trị dự kiến trong tương lai (Customer Lifetime Value)

### 2.2. Ứng dụng trong Fintech/Banking

#### 2.2.1. Phân nhóm khách hàng
- VIP/Platinum: Khách hàng giá trị cao
- Gold: Khách hàng giá trị trung bình
- Silver/Bronze: Khách hàng giá trị thấp

#### 2.2.2. Chiến lược marketing
- **Retention**: Giữ chân khách hàng VIP
- **Win-back**: Thu hút lại khách hàng đã rời bỏ
- **Upsell**: Nâng cấp dịch vụ cho khách hàng hiện tại
- **Cross-sell**: Giới thiệu sản phẩm mới

#### 2.2.3. Quản lý rủi ro
- Đánh giá khả năng trả nợ
- Xác định hạn mức tín dụng phù hợp
- Phát hiện gian lận

## 3. RFM Analysis

### 3.1. RFM là gì?

RFM là phương pháp phân tích khách hàng dựa trên 3 chỉ số:

- **R (Recency)**: Khoảng thời gian từ lần giao dịch cuối cùng đến hiện tại
- **F (Frequency)**: Số lần giao dịch trong một khoảng thời gian
- **M (Monetary)**: Tổng giá trị giao dịch trong khoảng thời gian đó

### 3.2. Logic hành vi của RFM

#### 3.2.1. Recency (R)
- **R cao** (giao dịch gần đây): Khách hàng đang hoạt động, có khả năng tiếp tục sử dụng
- **R thấp** (lâu không giao dịch): Khách hàng có thể đã churn hoặc không còn quan tâm

#### 3.2.2. Frequency (F)
- **F cao**: Khách hàng trung thành, thường xuyên sử dụng dịch vụ
- **F thấp**: Khách hàng ít sử dụng, có thể cần kích thích

#### 3.2.3. Monetary (M)
- **M cao**: Khách hàng có giá trị cao, mang lại nhiều doanh thu
- **M thấp**: Khách hàng có giá trị thấp, cần chiến lược tăng giá trị

### 3.3. Cách tính RFM

#### 3.3.1. Dữ liệu cần có
- **Customer ID**: Mã định danh khách hàng
- **Transaction Date**: Ngày giao dịch
- **Transaction Amount**: Giá trị giao dịch

#### 3.3.2. Tính Recency (R)
```python
# R = Số ngày từ lần giao dịch cuối đến ngày phân tích
# R càng nhỏ = giao dịch càng gần đây (tốt hơn)
recency = (analysis_date - last_transaction_date).days
```

#### 3.3.3. Tính Frequency (F)
```python
# F = Số lần giao dịch trong khoảng thời gian quan sát
frequency = count(transactions)
```

#### 3.3.4. Tính Monetary (M)
```python
# M = Tổng giá trị giao dịch trong khoảng thời gian
monetary = sum(transaction_amounts)
```

### 3.4. Phân nhóm RFM

#### 3.4.1. Phương pháp Quintile (5 nhóm)
Chia khách hàng thành 5 nhóm theo mỗi chỉ số:
- **5**: Top 20% (tốt nhất)
- **4**: 20-40%
- **3**: 40-60%
- **2**: 60-80%
- **1**: Bottom 20% (kém nhất)

**Lưu ý**: Với Recency, giá trị nhỏ hơn = tốt hơn, nên cần đảo ngược thứ tự.

#### 3.4.2. RFM Score

Kết hợp 3 chỉ số thành 1 score:

$$RFM\_Score = R \times 100 + F \times 10 + M$$

Trong đó:
- $R$: Recency score (1-5)
- $F$: Frequency score (1-5)
- $M$: Monetary score (1-5)

**Ví dụ:**
- $RFM\_Score = 555$ → Khách hàng tốt nhất ($R=5, F=5, M=5$)
- $RFM\_Score = 111$ → Khách hàng kém nhất ($R=1, F=1, M=1$)
- $RFM\_Score = 451$ → $R=4$ (gần đây), $F=5$ (nhiều giao dịch), $M=1$ (giá trị thấp)

#### 3.4.3. Phân nhóm khách hàng theo RFM

| RFM Score | Nhóm | Đặc điểm | Chiến lược |
|-----------|------|----------|------------|
| 555, 554, 545, 544 | Champions | R cao, F cao, M cao | Giữ chân, VIP program |
| 543, 542, 541, 535, 534, 533, 532, 531 | Loyal Customers | F cao, M cao | Upsell, cross-sell |
| 525, 524, 523, 522, 521, 515, 514, 513, 512, 511 | Potential Loyalists | R cao, F trung bình | Tăng tần suất sử dụng |
| 455, 454, 445, 444, 435, 434, 433, 432, 431 | New Customers | R cao, F thấp | Onboarding, activation |
| 355, 354, 345, 344, 335, 334, 333, 332, 331 | Promising | R trung bình, F trung bình | Engagement campaigns |
| 255, 254, 245, 244, 235, 234, 233, 232, 231 | Need Attention | R thấp, F cao | Win-back campaigns |
| 155, 154, 145, 144, 135, 134, 133, 132, 131 | About to Sleep | R thấp, F trung bình | Re-engagement |
| 55, 54, 45, 44, 35, 34, 33, 32, 31 | At Risk | R thấp, F thấp | Win-back hoặc chấp nhận churn |
| 25, 24, 23, 22, 21, 15, 14, 13, 12, 11 | Lost | R rất thấp, F thấp | Chấp nhận churn hoặc win-back đặc biệt |

### 3.5. Ý nghĩa của RFM trong phân nhóm khách hàng

1. **Xác định khách hàng giá trị**: Champions, Loyal Customers
2. **Phát hiện khách hàng có nguy cơ churn**: At Risk, About to Sleep
3. **Tối ưu ngân sách marketing**: Tập trung vào nhóm có tiềm năng
4. **Cá nhân hóa dịch vụ**: Mỗi nhóm có chiến lược riêng
5. **Đo lường hiệu quả**: So sánh RFM trước và sau chiến dịch

## 4. Phân tích khách hàng theo hành vi lịch sử (Historical Value)

### 4.1. Historical Value Metrics

Ngoài RFM, có thể bổ sung các metrics khác:

- **Average Transaction Value**: Giá trị giao dịch trung bình
- **Transaction Velocity**: Tốc độ giao dịch (số giao dịch/tháng)
- **Product Diversity**: Số loại sản phẩm/dịch vụ đã sử dụng
- **Tenure**: Thời gian là khách hàng
- **Growth Rate**: Tốc độ tăng trưởng giá trị theo thời gian

### 4.2. Time-based Analysis

- **Trend Analysis**: Xu hướng giá trị theo thời gian
- **Seasonality**: Tính chu kỳ trong hành vi
- **Lifecycle Stage**: Giai đoạn trong vòng đời khách hàng

### 4.3. Cohort Analysis

Phân tích theo cohort (nhóm khách hàng cùng thời điểm):
- Cohort theo tháng đăng ký
- Retention rate theo cohort
- Value evolution theo cohort

## 5. Chuẩn hóa Feature

### 5.1. Tại sao cần chuẩn hóa?

- RFM có đơn vị khác nhau (ngày, số lần, tiền)
- Cần chuẩn hóa để so sánh và kết hợp
- Tránh bias về đơn vị đo

### 5.2. Phương pháp chuẩn hóa

#### 5.2.1. Min-Max Scaling

$$X_{normalized} = \frac{X - X_{min}}{X_{max} - X_{min}}$$

**Kết quả:** Giá trị được scale về khoảng $[0, 1]$
- $X_{min}$: Giá trị nhỏ nhất trong dataset
- $X_{max}$: Giá trị lớn nhất trong dataset

#### 5.2.2. Z-score Normalization (Standardization)

$$X_{normalized} = \frac{X - \mu}{\sigma}$$

Trong đó:
- $\mu$: Mean (trung bình) của $X$
- $\sigma$: Standard deviation (độ lệch chuẩn) của $X$

**Kết quả:** Mean = 0, Std = 1 (standard normal distribution)

#### 5.2.3. Robust Scaling

$$X_{normalized} = \frac{X - \text{median}(X)}{IQR}$$

Trong đó:
- $\text{median}(X)$: Median (trung vị) của $X$
- $IQR$: Interquartile Range (khoảng tứ phân vị) = $Q_3 - Q_1$

**Ưu điểm:** Phù hợp khi có outliers (không bị ảnh hưởng bởi extreme values)

### 5.3. Chuẩn hóa RFM

- **Recency**: Càng nhỏ càng tốt → Cần đảo ngược hoặc dùng negative
- **Frequency**: Càng lớn càng tốt → Giữ nguyên
- **Monetary**: Càng lớn càng tốt → Giữ nguyên

## 6. Visualization

### 6.1. RFM Heatmap

Heatmap hiển thị phân bố khách hàng theo R và F, với màu sắc thể hiện M trung bình.

### 6.2. Segment Profile

- Bar chart: So sánh R, F, M trung bình giữa các segment
- Pie chart: Tỷ lệ khách hàng trong mỗi segment
- Box plot: Phân bố giá trị trong mỗi segment

### 6.3. Customer Journey Visualization

- Sankey diagram: Chuyển đổi giữa các segment theo thời gian
- Timeline: Hành trình của khách hàng qua các segment

## 7. Best Practices

1. **Chọn khoảng thời gian phù hợp**: Thường 6-12 tháng
2. **Cập nhật định kỳ**: RFM nên được tính lại hàng tháng/quý
3. **Kết hợp với domain knowledge**: Hiểu rõ ngữ cảnh fintech/banking
4. **Validate với business**: Đảm bảo segment có ý nghĩa thực tế
5. **Monitor và điều chỉnh**: Theo dõi hiệu quả và cải thiện
