

# 📊 Store Sales & Profit Forecasting Analysis
### 零售家具超市銷售與利潤預測分析 

本專案旨在解決零售業中「利潤預測」的核心問題。我們透過 **Spline Regression（樣條迴歸）** 捕捉折扣與利潤之間的複雜非線性關係，並引入 **L1 Regularization（L1 正則化）** 進行特徵選擇，最後透過 **Prediction Intervals** 進行風險評估。

---

## 🔍 資料說明與處理 (Data Overview)
* **資料來源**：[Kaggle - Store Sales Forecasting Dataset](https://www.kaggle.com/datasets/tanayatipre/store-sales-forecasting-dataset)
* **目標變數**：`Profit` (利潤)
* **關鍵特徵**：
    * `Discount`：數值型，與利潤具顯著負相關 (-0.48)。
    * `Sub-Category`：類別型，包含 Furnishings, Chairs, Bookcases, Tables。
* **前處理**：
    * 移除重複值（`drop_duplicates`）。
    * 針對 `Sub-Category` 進行 **One-Hot Encoding** (Dummy Variables)。

---

## 🛠 建模方法 (Methodology)

### 1. Spline Regression (樣條迴歸)
為了捕捉非線性趨勢，我們針對 `Discount` 設定了 5 個 Knots (10%, 25%, 50%, 75%, 90% 百分位數)，並建構 **Hinge Features**: $max(0, x - knot)$。
* **優化目標**：最小化 MAE (Mean Absolute Error)。
* **求解工具**：Gurobi Optimizer。

### 2. L1 Regularization (模型壓縮)
透過約束參數總和 $\sum |\beta_j| \le B$，觀察不同預算 $B$ 對模型精準度的影響。

### 3. Risk Analysis (風險分析)
* 使用 **Laplace Distribution** 模擬殘差。
* 建立 **Heteroscedasticity（異質性變異數）** 模型，模擬不同折扣下的誤差波動。

---

## 📈 實驗結果 (Results)

### L1 Regularization 比較
| Budget (B) | CV-MAE | 結論 |
| :--- | :--- | :--- |
| **5** | 65.0573 | 模型過於簡化，誤差最大 |
| **20** | 63.5910 | 預測表現開始明顯改善 |
| **50** | 61.3000 | 涵蓋關鍵資訊，邊際效益開始遞減 |
| **100** | 59.3104 | 接近無限制模型，預測效果最佳 |
| **Unconstrained** | **53.65** | **基準效能** |

### 預測區間表現 (Prediction Interval)
我們發現平均 **Coverage Rate 為 63.77%**。雖然低於理論的 90%，但顯示了模型在不同資料分布（Fold）下的穩健性差異。其中 **Fold 5** 表現最優（MSIS 僅 32.44），驗證了異質性變異數建模在穩定數據下的可行性。

---

## 💡 專案洞察 (Insights)
1.  **非線性折扣效應**：模型顯示小幅折扣會劇烈壓縮利潤，但當折扣極高時（90th percentile 以上），利潤反而有拉升趨勢，可能與清庫存或組合銷售策略有關。
2.  **品類利潤差異**：`Chairs` 類別在模型中展現出最強的獲利潛力，而 `Furnishings` 則相對較低。
3.  **決策建議**：企業應將 L1 預算設定在 $30 \sim 50$ 之間，以獲得兼具「解釋性」與「預測力」的平衡模型。

---

## 📚 參考資料
Tipre, T. (2024). Store sales forecasting dataset. Kaggle.

