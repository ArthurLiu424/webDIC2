# 50 Startups 多元線性回歸探索器

> 物聯網數據分析與應用 DIC2：多元線性回歸（Multiple Linear Regression）與特徵選擇

用 **50 Startups** 資料集，根據新創公司的各項支出和所在州來預測 **Profit（利潤）**。網站比較 5 種特徵選擇方法，找出「要用幾個變數、用哪幾個變數」做出來的模型最好，也就是在準確度和簡單之間取得平衡（Parsimony）。

## 網站內容

| 區塊 | 說明 |
| --- | --- |
| 演算法切換 | Lasso、Forward SFS、RFE、ElasticNet、Exhaustive Search，也可以「比較全部 5 種方法」 |
| 特徵數 k = 1～5 | 切換模型使用的變數數量，自動標出最佳 k（★）和過擬合的完整模型（⚠） |
| 指標卡片 | 測試集 RMSE、R²、Adjusted R²、MAE、F 統計量 |
| RMSE / R² 折線圖 | 顯示誤差隨變數數量的變化，點擊資料點可以切換 k |
| 回歸方程式 | 目前模型的 OLS 方程式 |
| 係數與統計檢定 | 估計值、標準誤、t 值、p 值、95% 信賴區間、VIF（共線性） |
| 即時利潤預測器 | 輸入研發、行銷、管理支出和州別，即時算出預測利潤 |
| 殘差診斷 | 測試集 10 筆樣本依 \|殘差\| 排序，標出高殘差的樣本 |

## 資料集

[50_Startups.csv](https://raw.githubusercontent.com/Avik-Jain/100-Days-Of-ML-Code/master/datasets/50_Startups.csv)：共 50 筆。

| 欄位 | 說明 |
| --- | --- |
| R&D Spend | 研發支出 |
| Administration | 管理支出 |
| Marketing Spend | 行銷支出 |
| State | 所在州（New York / California / Florida） |
| Profit | 利潤（目標變數） |

## 分析流程（CRISP-DM）

1. **Business Understanding**：哪些支出真正影響新創公司的利潤？
2. **Data Understanding**：50 筆資料，3 個數值變數、1 個類別變數
3. **Data Preparation**
   - `State` 做 One-hot encoding，以 California 為基準組，產生 `State_Florida`、`State_New York`，共 5 個特徵
   - 用固定亂數種子（seed = 42）切出訓練集 40 筆、測試集 10 筆
4. **Modeling**：用 5 種方法決定每個 k 要選哪些特徵，再用 OLS 在訓練集上擬合
   - **Lasso**：L1 正則化路徑，依變數「進入模型」的先後排序
   - **ElasticNet**：L1 + L2（l1_ratio = 0.5），排序方式同上
   - **RFE**：遞迴刪除標準化係數絕對值最小的變數
   - **Forward SFS**：每一步加入讓 5-fold CV RMSE 最低的變數
   - **Exhaustive Search**：每個 k 窮舉所有組合，選 CV RMSE 最低的
5. **Evaluation**：在測試集上計算 RMSE、MAE、R²、Adjusted R²；在訓練集上做 t 檢定、F 檢定，並計算 VIF
6. **Deployment**：做成互動網頁

## 主要結果

| k | 使用的特徵（Lasso） | 測試 RMSE | 測試 R² |
| :-: | --- | --: | --: |
| 1 | R&D Spend | $9,850.21 | 0.9399 |
| **2 ★** | **R&D Spend, Marketing Spend** | **$9,304.12** | **0.9464** |
| 3 | + State_Florida | $9,437.23 | 0.9448 |
| 4 | + State_New York | $9,578.31 | 0.9432 |
| 5 ⚠ | + Administration（全部） | $9,668.04 | 0.9421 |

- **最佳模型是 k = 2**：5 種方法都選出 **R&D Spend + Marketing Spend**，測試誤差最低。
- **R&D Spend 是最重要的變數**：只用它一個，R² 就有 0.94。
- **全部變數放進去反而更差**：Administration 和 State 對利潤沒有顯著影響，加進來只會增加變異，k = 5 的 RMSE 比 k = 2 高約 $364，這就是**過擬合**。
- 最佳模型的方程式：

  ```
  Profit = 46,173.60 + 0.8145 × (R&D Spend) + 0.0247 × (Marketing Spend)
  ```

  意思是研發支出每多 $1，利潤大約增加 $0.81。兩個變數的 VIF 都是 1.90（< 5），沒有共線性問題。

## 使用方式

- **線上瀏覽**：`https://<你的帳號>.github.io/<repo 名稱>/`
- **本機開啟**：直接用瀏覽器打開 `index.html`

## 檔案結構

```
├── index.html        # 網頁（版面、樣式、所有模型計算）
├── data.js           # 50_Startups.csv 的內嵌版本，讓網頁直接開啟也能讀到資料
└── 50_Startups.csv   # 原始資料集
```

## 技術

- 純 HTML / CSS / JavaScript，不需要後端
- 所有計算（OLS、座標下降法 Lasso / ElasticNet、RFE、SFS、窮舉搜尋、t / F 分佈）都在瀏覽器中即時執行
- 圖表：[Chart.js](https://www.chartjs.org/)；圖示：Font Awesome

## 參考資料

- 版面參考：[AIoT-DIC2-MultipleLR](https://huanchen1107.github.io/AIoT-DIC2-MultipleLR/)、[StatsUnlock Multiple Linear Regression Calculator](https://statsunlock.com/multiple-linear-regression-calculator/)
- 資料集：[Avik Jain – 100 Days Of ML Code](https://github.com/Avik-Jain/100-Days-Of-ML-Code)
