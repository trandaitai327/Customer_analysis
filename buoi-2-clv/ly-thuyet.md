# Lý thuyết: BG-NBD & Gamma-Gamma → Predictive CLV

## 📑 Mục lục

1. [Giới thiệu mô hình dự báo CLV (Predictive Lifetime Value)](#1-giới-thiệu-mô-hình-dự-báo-clv-predictive-lifetime-value)
   - 1.1. CLV là gì?
   - 1.2. Tại sao cần Predictive CLV?
   - 1.3. Công thức CLV cơ bản
   - 1.4. Phương pháp tính CLV

2. [BG-NBD Model](#2-bg-nbd-model)
   - 2.1. Tổng quan
   - 2.2. Logic "Alive/Dead" Customer
   - 2.3. Giả định và công thức của BG-NBD
   - 2.4. Tham số của BG-NBD
   - 2.5. Input của BG-NBD
   - 2.6. Output của BG-NBD
   - 2.7. Công thức BG-NBD
   - 2.8. Ví dụ minh họa: Tại sao cần cả NBD và BG?
   - 2.9. Ước lượng tham số

3. [Gamma-Gamma Model](#3-gamma-gamma-model)
   - 3.1. Tổng quan
   - 3.2. Giả định
   - 3.3. Input của Gamma-Gamma
   - 3.4. Output của Gamma-Gamma
   - 3.5. Công thức Gamma-Gamma
   - 3.6. Ước lượng tham số

4. [Kết hợp BG-NBD và Gamma-Gamma để tính CLV](#4-kết-hợp-bg-nbd-và-gamma-gamma-để-tính-clv)
   - 4.1. Công thức CLV
   - 4.2. CLV trong khoảng thời gian t
   - 4.3. CLV với discount rate
   - 4.4. Quy trình tính CLV

5. [Ứng dụng CLV trong Cross-sell, Up-sell, Ưu tiên khách hàng](#5-ứng-dụng-clv-trong-cross-sell-up-sell-ưu-tiên-khách-hàng)
   - 5.1. Cross-sell
   - 5.2. Up-sell
   - 5.3. Ưu tiên khách hàng
   - 5.4. Tối ưu ngân sách marketing
   - 5.5. Customer Acquisition Cost (CAC) vs CLV

6. [Ưu và nhược điểm của BG-NBD + Gamma-Gamma](#6-ưu-và-nhược-điểm-của-bg-nbd--gamma-gamma)
   - 6.1. Ưu điểm
   - 6.2. Nhược điểm

7. [Best Practices](#7-best-practices)

8. [Thư viện Python](#8-thư-viện-python)

---

## 1. Giới thiệu mô hình dự báo CLV (Predictive Lifetime Value)

### 1.1. CLV là gì?

**Customer Lifetime Value (CLV)** là tổng giá trị dự kiến mà một khách hàng sẽ mang lại cho doanh nghiệp trong suốt mối quan hệ với họ.

### 1.2. Tại sao cần Predictive CLV?

#### 1.2.1. Hạn chế của Historical Value
- Chỉ phản ánh quá khứ, không dự đoán tương lai
- Không tính đến khả năng churn
- Không ước lượng được số giao dịch và giá trị tương lai

#### 1.2.2. Lợi ích của Predictive CLV
- **Dự đoán tương lai**: Biết được giá trị khách hàng sẽ mang lại
- **Tối ưu ngân sách**: Đầu tư vào khách hàng có CLV cao
- **Cá nhân hóa**: Chiến lược phù hợp với từng khách hàng
- **ROI tính toán**: So sánh chi phí acquisition với CLV

### 1.3. Công thức CLV cơ bản

**Công thức đơn giản:**

$$CLV = \text{Average Transaction Value} \times \text{Number of Transactions} \times \text{Customer Lifespan}$$

**Công thức với discount rate:**

$$CLV = \sum_{t=1}^{T} \frac{Revenue_t}{(1 + r)^t}$$

Trong đó:
- $Revenue_t$: Doanh thu tại thời điểm $t$
- $r$: Discount rate (tỷ lệ chiết khấu)
- $t$: Thời gian (tháng/năm)
- $T$: Thời gian tổng cộng

### 1.4. Phương pháp tính CLV

1. **Historical CLV**: Dựa trên dữ liệu quá khứ
2. **Predictive CLV**: Sử dụng mô hình thống kê/machine learning
   - BG-NBD + Gamma-Gamma (probabilistic)
   - Machine Learning (regression, neural networks)

## 2. BG-NBD Model

### 2.1. Tổng quan

**BG-NBD (Beta Geometric / Negative Binomial Distribution)** là mô hình probabilistic để dự đoán số giao dịch tương lai của khách hàng.

**BG-NBD = NBD (Negative Binomial Distribution) + BG (Beta Geometric)**

Mô hình này kết hợp **2 thành phần chính** để giải quyết 2 vấn đề riêng biệt:

1. **NBD (Negative Binomial Distribution)**: Mô hình hóa **số giao dịch** khi khách hàng còn "alive"
2. **BG (Beta Geometric)**: Mô hình hóa **quá trình churn** (khách hàng trở thành "dead")

### 2.1.1. Tại sao cần cả 2 thành phần?

#### Vấn đề thực tế:

Khi quan sát dữ liệu giao dịch, chúng ta thấy:
- Một số khách hàng giao dịch thường xuyên (frequency cao)
- Một số khách hàng giao dịch ít (frequency thấp)
- Một số khách hàng đã lâu không giao dịch

**Câu hỏi quan trọng**: Khách hàng không giao dịch là do:
- **Lý do 1**: Họ có tần suất giao dịch thấp (λ thấp) nhưng vẫn còn "alive" → Sẽ giao dịch trong tương lai
- **Lý do 2**: Họ đã churn (trở thành "dead") → Sẽ không bao giờ giao dịch nữa

**→ Cần phân biệt được 2 trường hợp này!**

#### Giải pháp: Kết hợp NBD + BG

**NBD (Negative Binomial Distribution)** - Mô hình số giao dịch:
- **Mục đích**: Mô hình hóa sự khác biệt về **tần suất giao dịch** giữa các khách hàng
- **Giả định**: Khách hàng còn "alive" sẽ giao dịch theo quá trình Poisson với rate λ
- **λ khác nhau** giữa các khách hàng → Mô hình bằng Gamma distribution
- **Kết quả**: Phân phối số giao dịch khi khách hàng còn "alive" là **Negative Binomial**

**BG (Beta Geometric)** - Mô hình quá trình churn:
- **Mục đích**: Mô hình hóa **thời điểm churn** của khách hàng
- **Giả định**: Sau mỗi giao dịch, khách hàng có xác suất $p$ để churn
- **$p$ khác nhau** giữa các khách hàng → Mô hình bằng Beta distribution
- **Kết quả**: Số giao dịch trước khi churn tuân theo **Geometric distribution** với $p$ từ Beta

#### Tại sao không thể dùng riêng lẻ?

**Nếu chỉ dùng NBD:**
- ❌ Giả định tất cả khách hàng đều còn "alive"
- ❌ Không thể phân biệt khách hàng "alive nhưng ít giao dịch" vs "đã churn"
- ❌ Dự đoán sẽ sai cho khách hàng đã churn (sẽ dự đoán họ vẫn giao dịch)

**Nếu chỉ dùng BG:**
- ❌ Chỉ mô hình được quá trình churn, không mô hình được tần suất giao dịch
- ❌ Không thể dự đoán số giao dịch tương lai (chỉ biết khi nào churn)
- ❌ Bỏ qua sự khác biệt về hành vi giao dịch giữa các khách hàng

**Kết hợp NBD + BG:**
- ✅ Mô hình được cả tần suất giao dịch (NBD) và quá trình churn (BG)
- ✅ Phân biệt được khách hàng "alive nhưng ít giao dịch" vs "đã churn"
- ✅ Dự đoán chính xác số giao dịch tương lai, tính cả khả năng churn

### 2.1.2. Cơ chế hoạt động của BG-NBD

**Quá trình mô hình hóa:**

```
1. Khách hàng bắt đầu với:
   - Transaction rate: λ ~ Gamma(r, α)  [từ NBD]
   - Churn probability: p ~ Beta(a, b)  [từ BG]

2. Khi khách hàng còn "alive":
   - Giao dịch xảy ra theo Poisson process với rate λ
   - Sau mỗi giao dịch, có xác suất p để churn

3. Nếu churn (trở thành "dead"):
   - Không còn giao dịch nữa

4. Quan sát được:
   - x: Số giao dịch trong quá khứ
   - t_x: Thời gian từ lần đầu đến lần cuối
   - T: Tổng thời gian quan sát

5. Cần ước lượng:
   - P(alive): Xác suất còn "alive" tại thời điểm T
   - E[transactions]: Số giao dịch dự kiến trong tương lai
```

**Visualization minh họa:**

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import nbinom, geom

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# ===== NBD: Negative Binomial Distribution =====
# NBD mô hình số giao dịch khi còn "alive"
# NBD = Gamma-Poisson mixture
n = np.arange(0, 20)
for r, p_nbd in [(2, 0.3), (3, 0.4), (5, 0.5)]:
    # Negative Binomial: số lần thất bại trước khi đạt r lần thành công
    # Ở đây: số giao dịch = r + số lần "không giao dịch"
    axes[0, 0].plot(n, nbinom.pmf(n, r, p_nbd), 
                   marker='o', label=f'r={r}, p={p_nbd}', linewidth=2)
axes[0, 0].set_xlabel('Number of Transactions', fontsize=12)
axes[0, 0].set_ylabel('Probability', fontsize=12)
axes[0, 0].set_title('NBD: Distribution of Transactions (when alive)', 
                     fontsize=14, fontweight='bold')
axes[0, 0].legend(fontsize=10)
axes[0, 0].grid(True, alpha=0.3)

# ===== BG: Beta Geometric =====
# Geometric distribution mô hình số giao dịch trước khi churn
# p khác nhau → Beta distribution
k = np.arange(1, 20)  # Số giao dịch trước khi churn
for p_geom in [0.1, 0.2, 0.3]:
    axes[0, 1].plot(k, geom.pmf(k, p_geom), 
                   marker='s', label=f'p={p_geom}', linewidth=2)
axes[0, 1].set_xlabel('Number of Transactions Before Churn', fontsize=12)
axes[0, 1].set_ylabel('Probability', fontsize=12)
axes[0, 1].set_title('Geometric: Transactions Before Churn', 
                     fontsize=14, fontweight='bold')
axes[0, 1].legend(fontsize=10)
axes[0, 1].grid(True, alpha=0.3)

# ===== So sánh: Chỉ NBD vs BG-NBD =====
# Scenario: Khách hàng có 0 giao dịch trong 90 ngày
time_points = np.arange(0, 180, 1)

# Chỉ dùng NBD (giả định tất cả đều alive)
lambda_low = 0.01  # Tần suất thấp
prob_alive_nbd_only = np.exp(-lambda_low * time_points)  # Xác suất ít nhất 1 giao dịch
expected_trans_nbd = lambda_low * time_points

# BG-NBD (tính cả khả năng churn)
# Giả sử p = 0.3 (30% churn sau mỗi giao dịch)
p_churn = 0.3
# Nếu đã churn → không giao dịch
# Nếu còn alive với λ thấp → ít giao dịch
prob_alive_bgnbd = 0.7  # Giả sử 70% còn alive
expected_trans_bgnbd = prob_alive_bgnbd * lambda_low * time_points

axes[1, 0].plot(time_points, expected_trans_nbd, 
               label='NBD Only (assumes all alive)', linewidth=2, linestyle='--')
axes[1, 0].plot(time_points, expected_trans_bgnbd, 
               label='BG-NBD (accounts for churn)', linewidth=2)
axes[1, 0].set_xlabel('Time (days)', fontsize=12)
axes[1, 0].set_ylabel('Expected Transactions', fontsize=12)
axes[1, 0].set_title('Comparison: NBD Only vs BG-NBD', 
                     fontsize=14, fontweight='bold')
axes[1, 0].legend(fontsize=10)
axes[1, 0].grid(True, alpha=0.3)

# ===== P(alive) theo thời gian =====
# Khách hàng không giao dịch trong t ngày
t_no_transaction = np.arange(0, 180, 1)

# NBD only: P(alive) = 1 (luôn giả định còn alive)
prob_alive_nbd = np.ones_like(t_no_transaction)

# BG-NBD: P(alive) giảm dần nếu không có giao dịch
# Công thức đơn giản hóa: P(alive) giảm theo thời gian
prob_alive_bgnbd_time = np.exp(-0.02 * t_no_transaction)  # Giảm dần

axes[1, 1].plot(t_no_transaction, prob_alive_nbd, 
               label='NBD Only (P(alive) = 1)', linewidth=2, linestyle='--')
axes[1, 1].plot(t_no_transaction, prob_alive_bgnbd_time, 
               label='BG-NBD (P(alive) decreases)', linewidth=2)
axes[1, 1].set_xlabel('Days Without Transaction', fontsize=12)
axes[1, 1].set_ylabel('P(alive)', fontsize=12)
axes[1, 1].set_title('P(alive) Over Time (No Transactions)', 
                     fontsize=14, fontweight='bold')
axes[1, 1].legend(fontsize=10)
axes[1, 1].grid(True, alpha=0.3)
axes[1, 1].set_ylim(0, 1.1)

plt.tight_layout()
plt.savefig('bgnbd_components.png', dpi=300, bbox_inches='tight')
plt.show()
```

**Giải thích hình ảnh:**

1. **NBD (trên trái)**: Phân phối số giao dịch khi khách hàng còn "alive"
   - Mô tả sự khác biệt về tần suất giao dịch
   - Khách hàng có λ cao → nhiều giao dịch hơn

2. **Geometric (trên phải)**: Phân phối số giao dịch trước khi churn
   - Mô tả quá trình churn
   - Khách hàng có p cao → churn sớm hơn

3. **So sánh NBD vs BG-NBD (dưới trái)**: 
   - NBD only: Luôn giả định khách hàng còn alive → dự đoán quá cao
   - BG-NBD: Tính cả khả năng churn → dự đoán chính xác hơn

4. **P(alive) theo thời gian (dưới phải)**:
   - NBD only: P(alive) = 1 (luôn giả định còn alive)
   - BG-NBD: P(alive) giảm dần nếu không có giao dịch → realistic hơn

### 2.2. Logic "Alive/Dead" Customer

#### 2.2.1. Khái niệm
- **Alive**: Khách hàng vẫn đang hoạt động, có khả năng thực hiện giao dịch
- **Dead**: Khách hàng đã churn, không còn thực hiện giao dịch

#### 2.2.2. Vấn đề
- Không thể quan sát trực tiếp trạng thái "alive/dead"
- Khách hàng không giao dịch có thể:
  - Đang "alive" nhưng chưa có nhu cầu
  - Đã "dead" (churn)

#### 2.2.3. Giải pháp
BG-NBD ước lượng xác suất khách hàng còn "alive" dựa trên hành vi quá khứ.

### 2.3. Giả định và công thức của BG-NBD

#### 2.3.1. Công thức phân phối: Gamma và Beta

[Phần công thức chi tiết được trình bày ở dưới, xem 2.3.1.1 và 2.3.1.2]

#### 2.3.2. Giả định từ NBD (Negative Binomial)

1. **Khách hàng hoạt động với tần suất λ (lambda)**:
   - Khi còn "alive", khách hàng giao dịch theo **Poisson process** với rate $\lambda$
   - $\lambda$ khác nhau giữa các khách hàng → Mô hình bằng **Gamma distribution**: $\lambda \sim \text{Gamma}(r, \alpha)$
   - Mỗi khách hàng có $\lambda$ riêng (heterogeneity trong transaction rate)
   - **Kết quả**: Số giao dịch khi còn "alive" tuân theo **Negative Binomial Distribution**

**Tại sao Negative Binomial?**
- Poisson($\lambda$) với $\lambda \sim \text{Gamma}(r, \alpha)$ → Negative Binomial
- Negative Binomial mô hình được sự khác biệt về tần suất giữa các khách hàng
- Phù hợp với dữ liệu thực tế: đa số khách hàng giao dịch ít, một số ít giao dịch rất nhiều

#### 2.3.3. Giả định từ BG (Beta Geometric)

2. **Sau mỗi giao dịch, khách hàng có thể churn**:
   - Xác suất churn sau mỗi giao dịch: $p$
   - $p$ khác nhau giữa các khách hàng → Mô hình bằng **Beta distribution**: $p \sim \text{Beta}(a, b)$
   - Mỗi khách hàng có $p$ riêng (heterogeneity trong churn probability)
   - **Kết quả**: Số giao dịch trước khi churn tuân theo **Geometric Distribution** với $p$ từ Beta

**Tại sao Geometric?**
- Geometric distribution mô hình số lần thử (giao dịch) trước khi thành công (churn)
- Phù hợp với giả định: sau mỗi giao dịch có xác suất $p$ để churn
- Beta-Geometric: $p$ khác nhau giữa khách hàng → mô hình được heterogeneity

#### 2.3.4. Giả định kết hợp

3. **Khách hàng "dead" không còn giao dịch**
   - Một khi đã churn, khách hàng không bao giờ giao dịch nữa
   - Đây là giả định quan trọng để phân biệt "alive nhưng ít giao dịch" vs "đã churn"

4. **$\lambda$ và $p$ độc lập giữa các khách hàng**
   - Tần suất giao dịch ($\lambda$) và xác suất churn ($p$) là độc lập
   - Một khách hàng có thể có $\lambda$ cao (giao dịch nhiều) nhưng $p$ cũng cao (dễ churn)
   - Hoặc $\lambda$ thấp (giao dịch ít) nhưng $p$ thấp (ít churn)

**Tóm tắt vai trò:**

| Thành phần | Vai trò | Mô hình hóa | Phân phối |
|------------|---------|------------|-----------|
| **NBD** | Tần suất giao dịch | Sự khác biệt về transaction rate | Negative Binomial |
| **BG** | Quá trình churn | Sự khác biệt về churn probability | Beta-Geometric |
| **Kết hợp** | Dự đoán chính xác | Cả tần suất và churn | BG-NBD |

#### 2.3.1. Công thức phân phối: Gamma và Beta

##### 2.3.1.1. Công thức Gamma Distribution

**Gamma Distribution** được sử dụng để mô hình hóa transaction rate $\lambda$:

**Probability Density Function (PDF):**

$$f(\lambda \mid r, \alpha) = \frac{\alpha^r}{\Gamma(r)} \lambda^{r-1} e^{-\alpha \lambda}, \quad \lambda > 0$$

Trong đó:
- $\lambda$: Transaction rate (biến ngẫu nhiên)
- $r > 0$: Shape parameter (tham số hình dạng)
- $\alpha > 0$: Rate parameter (tham số tỷ lệ)
- $\Gamma(r)$: Gamma function = $\int_0^{\infty} t^{r-1} e^{-t} dt$

**Cumulative Distribution Function (CDF):**

$$F(\lambda \mid r, \alpha) = \frac{\gamma(r, \alpha \lambda)}{\Gamma(r)}$$

Trong đó $\gamma(r, \alpha \lambda)$ là lower incomplete gamma function.

**Mean và Variance:**

$$E[\lambda] = \frac{r}{\alpha}$$

$$\text{Var}(\lambda) = \frac{r}{\alpha^2}$$

**Mode (giá trị có xác suất cao nhất):**

$$\text{Mode}(\lambda) = \frac{r-1}{\alpha}, \quad \text{nếu } r \geq 1$$

**Tính chất:**
- **Support**: $\lambda \in (0, \infty)$ → Phù hợp với transaction rate (luôn dương)
- **Shape**: 
  - $r < 1$: Lệch phải mạnh, giảm dần từ 0
  - $r = 1$: Exponential distribution
  - $r > 1$: Có peak, lệch phải
  - $r \to \infty$: Tiến về Normal distribution
- **Rate parameter $\alpha$**: 
  - $\alpha$ lớn → Phân phối tập trung về 0 (nhiều khách hàng có $\lambda$ thấp)
  - $\alpha$ nhỏ → Phân phối rải rộng hơn (nhiều khách hàng có $\lambda$ cao)

**Ký hiệu:** $\lambda \sim \text{Gamma}(r, \alpha)$

##### 2.3.1.2. Công thức Beta Distribution

**Beta Distribution** được sử dụng để mô hình hóa churn probability $p$:

**Probability Density Function (PDF):**

$$f(p \mid a, b) = \frac{p^{a-1}(1-p)^{b-1}}{B(a, b)}, \quad 0 \leq p \leq 1$$

Trong đó:
- $p$: Churn probability (biến ngẫu nhiên)
- $a > 0$: Shape parameter 1
- $b > 0$: Shape parameter 2
- $B(a, b)$: Beta function = $\frac{\Gamma(a)\Gamma(b)}{\Gamma(a+b)} = \int_0^1 t^{a-1}(1-t)^{b-1} dt$

**Cumulative Distribution Function (CDF):**

$$F(p \mid a, b) = \frac{B(p; a, b)}{B(a, b)} = I_p(a, b)$$

Trong đó $I_p(a, b)$ là regularized incomplete beta function.

**Mean và Variance:**

$$E[p] = \frac{a}{a + b}$$

$$\text{Var}(p) = \frac{ab}{(a+b)^2(a+b+1)}$$

**Mode (giá trị có xác suất cao nhất):**

$$\text{Mode}(p) = \frac{a-1}{a+b-2}, \quad \text{nếu } a > 1, b > 1$$

**Tính chất:**
- **Support**: $p \in [0, 1]$ → Phù hợp với xác suất
- **Shape**:
  - $a = b = 1$: Uniform distribution trên [0, 1]
  - $a = b$: Phân phối đối xứng, mode tại $p = 0.5$
  - $a > b$: Lệch trái (nhiều khách hàng có $p$ thấp, ít churn)
  - $a < b$: Lệch phải (nhiều khách hàng có $p$ cao, dễ churn)
- **Interpretation**:
  - $a$: "Số lần thành công" (không churn)
  - $b$: "Số lần thất bại" (churn)
  - $a + b$: "Tổng số lần thử" (tổng số giao dịch)

**Ký hiệu:** $p \sim \text{Beta}(a, b)$

##### 2.3.1.3. Hình dạng phân phối

**Gamma Distribution (cho λ):**
- Hình dạng: Lệch phải (right-skewed), bắt đầu từ 0
- Tham số $r$ (shape): Càng lớn, phân phối càng đối xứng hơn
- Tham số $\alpha$ (rate): Càng lớn, phân phối càng tập trung về 0
- Ứng dụng: Mô hình hóa tần suất giao dịch (luôn ≥ 0)

**Beta Distribution (cho p):**
- Hình dạng: Phân phối trên khoảng [0, 1]
- Tham số $a, b$:
  - $a = b$: Phân phối đối xứng
  - $a > b$: Lệch trái (nhiều khách hàng có p thấp)
  - $a < b$: Lệch phải (nhiều khách hàng có p cao)
- Ứng dụng: Mô hình hóa xác suất churn (luôn trong [0, 1])

**Code để visualize phân phối:**

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import gamma, beta

# Tạo figure với 2 subplots
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# ===== Gamma Distribution (cho λ) =====
x_gamma = np.linspace(0, 10, 1000)
for r, alpha in [(1, 1), (2, 1), (3, 1), (2, 2)]:
    # scipy.stats.gamma dùng scale = 1/rate
    axes[0].plot(x_gamma, gamma.pdf(x_gamma, a=r, scale=1/alpha), 
                 label=f'r={r}, α={alpha}', linewidth=2)
axes[0].set_xlabel('λ (Transaction Rate)', fontsize=12)
axes[0].set_ylabel('Probability Density', fontsize=12)
axes[0].set_title('Gamma Distribution (for λ - Transaction Rate)', fontsize=14, fontweight='bold')
axes[0].legend(fontsize=10)
axes[0].grid(True, alpha=0.3)
axes[0].set_xlim(0, 10)

# ===== Beta Distribution (cho p) =====
x_beta = np.linspace(0, 1, 1000)
for a, b in [(1, 1), (2, 2), (2, 5), (5, 2)]:
    axes[1].plot(x_beta, beta.pdf(x_beta, a, b), 
                 label=f'a={a}, b={b}', linewidth=2)
axes[1].set_xlabel('p (Churn Probability)', fontsize=12)
axes[1].set_ylabel('Probability Density', fontsize=12)
axes[1].set_title('Beta Distribution (for p - Churn Probability)', fontsize=14, fontweight='bold')
axes[1].legend(fontsize=10)
axes[1].grid(True, alpha=0.3)
axes[1].set_xlim(0, 1)

plt.tight_layout()
plt.savefig('bg_nbd_distributions.png', dpi=300, bbox_inches='tight')
plt.show()
```

**Giải thích hình ảnh:**

**Gamma Distribution (trái):**
- Mô tả phân bố tần suất giao dịch $\lambda$ giữa các khách hàng
- Khách hàng có $\lambda$ cao → giao dịch thường xuyên hơn
- Phân phối lệch phải → đa số khách hàng có tần suất thấp, một số ít có tần suất rất cao

**Beta Distribution (phải):**
- Mô tả phân bố xác suất churn $p$ giữa các khách hàng
- Khách hàng có $p$ cao → dễ churn hơn
- Phân phối trên [0,1] → phù hợp với xác suất

##### 2.3.1.4. Ví dụ tính toán

**Ví dụ 1: Gamma Distribution**

Giả sử $\lambda \sim \text{Gamma}(r=2, \alpha=1)$:

- Mean: $E[\lambda] = \frac{2}{1} = 2$ (transaction rate trung bình = 2 giao dịch/tháng)
- Variance: $\text{Var}(\lambda) = \frac{2}{1^2} = 2$
- Mode: $\text{Mode}(\lambda) = \frac{2-1}{1} = 1$

**Ví dụ 2: Beta Distribution**

Giả sử $p \sim \text{Beta}(a=2, b=5)$:

- Mean: $E[p] = \frac{2}{2+5} = \frac{2}{7} \approx 0.286$ (xác suất churn trung bình = 28.6%)
- Variance: $\text{Var}(p) = \frac{2 \times 5}{(2+5)^2(2+5+1)} = \frac{10}{392} \approx 0.0255$
- Mode: $\text{Mode}(p) = \frac{2-1}{2+5-2} = \frac{1}{5} = 0.2$ (giá trị có xác suất cao nhất = 20%)

**Code tính toán:**

```python
import numpy as np
from scipy.stats import gamma, beta

# Gamma Distribution
r, alpha = 2, 1
lambda_dist = gamma(a=r, scale=1/alpha)
print(f"Gamma({r}, {alpha}):")
print(f"  Mean: {lambda_dist.mean():.2f}")
print(f"  Variance: {lambda_dist.var():.2f}")
print(f"  Mode: {(r-1)/alpha:.2f}")

# Beta Distribution
a, b = 2, 5
p_dist = beta(a=a, b=b)
print(f"\nBeta({a}, {b}):")
print(f"  Mean: {p_dist.mean():.4f}")
print(f"  Variance: {p_dist.var():.4f}")
print(f"  Mode: {(a-1)/(a+b-2):.4f}")
```

### 2.4. Tham số của BG-NBD

- **r, α**: Tham số của Gamma distribution cho λ
- **a, b**: Tham số của Beta distribution cho p

### 2.5. Input của BG-NBD

Với mỗi khách hàng, cần:
- **x**: Số lần giao dịch trong khoảng thời gian quan sát
- **tx**: Khoảng thời gian từ lần giao dịch đầu đến lần cuối
- **T**: Tổng thời gian quan sát (từ lần giao dịch đầu đến hiện tại)

### 2.6. Output của BG-NBD

- **P(alive)**: Xác suất khách hàng còn "alive"
- **E[transactions]**: Số giao dịch dự kiến trong tương lai

### 2.7. Công thức BG-NBD

#### 2.7.1. Xác suất khách hàng còn alive

$$P(\text{alive} \mid r, \alpha, a, b, x, t_x, T) = \frac{1}{1 + \frac{a}{b + x - 1} \cdot \left(\frac{\alpha + T}{\alpha + t_x}\right)^{r + x}}$$

Trong đó:
- $r, \alpha$: Tham số Gamma distribution cho $\lambda$
- $a, b$: Tham số Beta distribution cho $p$
- $x$: Số lần giao dịch trong quá khứ
- $t_x$: Khoảng thời gian từ lần giao dịch đầu đến lần cuối
- $T$: Tổng thời gian quan sát

#### 2.7.2. Số giao dịch dự kiến trong tương lai

$$E[\text{transactions trong } t \text{ periods tới}] = P(\text{alive}) \times \frac{r + x}{\alpha + T} \times t$$

Trong đó:
- $P(\text{alive})$: Xác suất khách hàng còn "alive" (từ công thức trên)
- $t$: Số periods (ngày/tháng) trong tương lai cần dự đoán

### 2.8. Ví dụ minh họa: Tại sao cần cả NBD và BG?

**Tình huống:** Có 2 khách hàng đều không giao dịch trong 60 ngày qua.

**Khách hàng A:**
- Trước đó: 10 giao dịch trong 6 tháng đầu
- Sau đó: 0 giao dịch trong 2 tháng gần đây
- **Câu hỏi**: Đã churn hay chỉ tạm thời không giao dịch?

**Khách hàng B:**
- Trước đó: 2 giao dịch trong 6 tháng đầu
- Sau đó: 0 giao dịch trong 2 tháng gần đây
- **Câu hỏi**: Đã churn hay chỉ có tần suất giao dịch thấp?

**Phân tích với BG-NBD:**

**Khách hàng A:**
- **NBD**: Có 10 giao dịch trước đó → $\lambda$ cao (giao dịch thường xuyên)
- **BG**: Không giao dịch 60 ngày → Có thể đã churn (vì trước đó giao dịch nhiều)
- **Kết luận**: P(alive) thấp → Có khả năng đã churn

**Khách hàng B:**
- **NBD**: Chỉ có 2 giao dịch trước đó → $\lambda$ thấp (giao dịch ít)
- **BG**: Không giao dịch 60 ngày → Có thể chỉ là do $\lambda$ thấp (chưa đến lúc giao dịch)
- **Kết luận**: P(alive) cao hơn → Có thể vẫn còn "alive", chỉ là tần suất thấp

**→ BG-NBD phân biệt được 2 trường hợp này!**

**Nếu chỉ dùng NBD:**
- Cả 2 khách hàng đều được giả định còn "alive"
- Không phân biệt được khách hàng A (có thể đã churn) vs B (có thể chỉ ít giao dịch)
- Dự đoán sẽ sai cho khách hàng A

### 2.8.1. Bảng so sánh NBD, BG, và BG-NBD

| Tiêu chí | NBD Only | BG Only | BG-NBD (Kết hợp) |
|----------|----------|---------|------------------|
| **Mô hình hóa** | Transaction rate | Churn process | Cả 2 |
| **Heterogeneity** | Có (λ khác nhau) | Có (p khác nhau) | Có (cả λ và p) |
| **Phân biệt "alive" vs "dead"** | ❌ Không | ✅ Có | ✅ Có |
| **Dự đoán số giao dịch** | ✅ Có (nhưng sai nếu đã churn) | ❌ Không | ✅ Có (chính xác) |
| **Dự đoán P(alive)** | ❌ Không (luôn = 1) | ✅ Có | ✅ Có |
| **Phù hợp với dữ liệu thực** | ⚠️ Một phần | ⚠️ Một phần | ✅ Tốt |
| **Ứng dụng** | Khi biết chắc tất cả còn alive | Khi chỉ quan tâm churn | Dự đoán CLV |

**Kết luận:**
- **NBD**: Tốt cho mô hình hóa transaction rate, nhưng không xử lý được churn
- **BG**: Tốt cho mô hình hóa churn, nhưng không mô hình được transaction rate
- **BG-NBD**: Kết hợp ưu điểm của cả 2 → Phù hợp nhất cho dự đoán CLV

### 2.9. Ước lượng tham số

Sử dụng Maximum Likelihood Estimation (MLE) để ước lượng $r, \alpha, a, b$ từ dữ liệu.

**Quy trình:**
1. Tính likelihood function dựa trên dữ liệu quan sát (x, t_x, T)
2. Tìm các tham số $r, \alpha, a, b$ để maximize likelihood
3. Sử dụng optimization algorithms (như BFGS, L-BFGS-B)
4. Validate với holdout data

## 3. Gamma-Gamma Model

### 3.1. Tổng quan

**Gamma-Gamma** là mô hình để ước lượng giá trị giao dịch trung bình tương lai của khách hàng.

### 3.2. Giả định

1. **Giá trị giao dịch của khách hàng i**: $Z_i$
2. **$Z_i$ tuân theo Gamma distribution**: $Z_i \sim \text{Gamma}(p, v)$
3. **Tham số $p$ tuân theo Gamma distribution**: $p \sim \text{Gamma}(q, \gamma)$
4. **$v$ cố định cho tất cả khách hàng**

**Visualization của Gamma-Gamma Model:**

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import gamma

# Tạo figure
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# ===== Gamma distribution cho Z_i (transaction value) =====
x_transaction = np.linspace(0, 2000, 1000)
# Ví dụ với các tham số khác nhau
for p, v in [(2, 0.01), (3, 0.01), (5, 0.01)]:
    # Gamma với shape=p, scale=1/v
    axes[0].plot(x_transaction, gamma.pdf(x_transaction, a=p, scale=1/v), 
                 label=f'p={p}, v={v}', linewidth=2)
axes[0].set_xlabel('Transaction Value Z', fontsize=12)
axes[0].set_ylabel('Probability Density', fontsize=12)
axes[0].set_title('Gamma Distribution for Transaction Value Z_i', 
                  fontsize=14, fontweight='bold')
axes[0].legend(fontsize=10)
axes[0].grid(True, alpha=0.3)
axes[0].set_xlim(0, 2000)

# ===== Gamma distribution cho parameter p =====
x_p = np.linspace(0, 10, 1000)
# Ví dụ với các tham số q, gamma khác nhau
for q, gamma_val in [(2, 1), (3, 1), (5, 1)]:
    axes[1].plot(x_p, gamma.pdf(x_p, a=q, scale=1/gamma_val), 
                 label=f'q={q}, γ={gamma_val}', linewidth=2)
axes[1].set_xlabel('Parameter p', fontsize=12)
axes[1].set_ylabel('Probability Density', fontsize=12)
axes[1].set_title('Gamma Distribution for Parameter p', 
                  fontsize=14, fontweight='bold')
axes[1].legend(fontsize=10)
axes[1].grid(True, alpha=0.3)
axes[1].set_xlim(0, 10)

plt.tight_layout()
plt.savefig('gamma_gamma_distributions.png', dpi=300, bbox_inches='tight')
plt.show()
```

**Giải thích:**
- **Trái**: Phân phối giá trị giao dịch $Z_i$ của từng khách hàng
- **Phải**: Phân phối tham số $p$ giữa các khách hàng (hierarchical structure)
- Gamma-Gamma cho phép mô hình hóa sự khác biệt về giá trị giao dịch giữa các khách hàng

### 3.3. Input của Gamma-Gamma

Với mỗi khách hàng:
- **mx**: Giá trị giao dịch trung bình trong quá khứ
- **x**: Số lần giao dịch

### 3.4. Output của Gamma-Gamma

- **E[Z]**: Giá trị giao dịch trung bình dự kiến trong tương lai

### 3.5. Công thức Gamma-Gamma

$$E[Z \mid q, \gamma, m_x, x] = \frac{\gamma + q \cdot m_x \cdot x}{\gamma + x}$$

Trong đó:
- $Z$: Giá trị giao dịch trung bình tương lai
- $q, \gamma$: Tham số của Gamma distribution
- $m_x$: Giá trị giao dịch trung bình trong quá khứ
- $x$: Số lần giao dịch trong quá khứ

### 3.6. Ước lượng tham số

Ước lượng `q, γ, v` từ dữ liệu bằng MLE.

## 4. Kết hợp BG-NBD và Gamma-Gamma để tính CLV

### 4.1. Công thức CLV

$$CLV = E[\text{transactions}] \times E[\text{average transaction value}]$$

Trong đó:
- $E[\text{transactions}]$: Số giao dịch dự kiến (từ BG-NBD)
- $E[\text{average transaction value}]$: Giá trị giao dịch trung bình dự kiến (từ Gamma-Gamma)

### 4.2. CLV trong khoảng thời gian t

$$CLV(t) = E[\text{transactions trong } t \text{ periods}] \times E[Z]$$

### 4.3. CLV với discount rate

$$CLV(t) = \sum_{i=1}^{t} \frac{E[\text{transactions trong period } i] \times E[Z]}{(1 + r)^i}$$

Trong đó:
- $r$: Discount rate (thường 0.1-0.15, tức 10-15% mỗi period)
- $i$: Thời kỳ (1, 2, 3, ..., t)
- $E[Z]$: Giá trị giao dịch trung bình dự kiến

### 4.4. Quy trình tính CLV

1. **Chuẩn bị dữ liệu**: Tính x, tx, T, mx cho mỗi khách hàng
2. **Fit BG-NBD**: Ước lượng r, α, a, b
3. **Fit Gamma-Gamma**: Ước lượng q, γ, v
4. **Tính P(alive)**: Từ BG-NBD
5. **Tính E[transactions]**: Từ BG-NBD
6. **Tính E[Z]**: Từ Gamma-Gamma
7. **Tính CLV**: E[transactions] × E[Z]

## 5. Ứng dụng CLV trong Cross-sell, Up-sell, Ưu tiên khách hàng

### 5.1. Cross-sell

**Mục tiêu**: Giới thiệu sản phẩm mới cho khách hàng hiện tại

**Chiến lược**:
- Tập trung vào khách hàng có CLV cao
- Khách hàng có P(alive) cao
- Khách hàng đã sử dụng nhiều sản phẩm

### 5.2. Up-sell

**Mục tiêu**: Nâng cấp dịch vụ cho khách hàng

**Chiến lược**:
- Khách hàng có CLV cao nhưng chưa sử dụng dịch vụ premium
- Khách hàng có E[Z] cao (giá trị giao dịch lớn)
- Khách hàng có frequency cao

### 5.3. Ưu tiên khách hàng

#### 5.3.1. Phân nhóm theo CLV

- **High CLV**: CLV > 75th percentile
- **Medium CLV**: 25th percentile < CLV < 75th percentile
- **Low CLV**: CLV < 25th percentile

#### 5.3.2. Chiến lược

| Nhóm | CLV | P(alive) | Chiến lược |
|------|-----|----------|------------|
| Champions | High | High | VIP program, retention |
| Potential | Medium | High | Upsell, cross-sell |
| At Risk | High | Low | Win-back campaign |
| Low Value | Low | Low | Cost-effective service |

### 5.4. Tối ưu ngân sách marketing

$$ROI = \frac{CLV \times \text{Conversion Rate}}{\text{Marketing Cost}}$$

Ưu tiên khách hàng có ROI cao nhất.

### 5.5. Customer Acquisition Cost (CAC) vs CLV

- **CAC**: Chi phí để có được khách hàng mới
- **CLV/CAC Ratio**: 
  - > 3: Tốt
  - 1-3: Cần cải thiện
  - < 1: Không bền vững

## 6. Ưu và nhược điểm của BG-NBD + Gamma-Gamma

### 6.1. Ưu điểm

- **Probabilistic**: Cung cấp xác suất, không chỉ điểm số
- **Interpretable**: Dễ hiểu và giải thích
- **Không cần nhiều features**: Chỉ cần transaction history
- **Handles churn**: Tự động xử lý khả năng churn
- **Proven**: Được sử dụng rộng rãi trong thực tế

### 6.2. Nhược điểm

- **Giả định phân phối**: Có thể không phù hợp với mọi loại dữ liệu
- **Không sử dụng features khác**: Chỉ dựa trên transaction history
- **Computational**: Cần optimize tham số
- **Assumes stationarity**: Giả định hành vi không đổi theo thời gian

## 7. Best Practices

1. **Chuẩn bị dữ liệu tốt**: Đảm bảo dữ liệu sạch và đầy đủ
2. **Chọn khoảng thời gian phù hợp**: Thường 6-12 tháng
3. **Validate mô hình**: So sánh dự đoán với thực tế
4. **Cập nhật định kỳ**: Retrain mô hình hàng quý
5. **Kết hợp với domain knowledge**: Hiểu rõ ngữ cảnh business
6. **Monitor performance**: Theo dõi accuracy của dự đoán

## 8. Thư viện Python

- **lifetimes**: Thư viện chuyên dụng cho BG-NBD và Gamma-Gamma
- **scipy**: Optimization và statistical functions
- **pandas, numpy**: Data manipulation
