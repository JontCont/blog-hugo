---
title: 關於我
slug: about
url: /about/
date: 2026-11-14T00:00:00+08:00
description: 全端軟體工程師 John Conte 的職涯故事與實戰經歷——從智慧製造 MES、17Life 電商、合作金庫與凱基銀行金融系統，到新光醫院 HIS 與 AIP 企業級生成式 AI 平台落地。
comments: false
---

## 👨‍💻 關於我 (About Me)

哈囉，我是 **John Conte（馬國勛）**，一名專注於 **C# / .NET 後端核心架構**、**Angular 全端開發**、**Azure 雲端服務** 與 **生成式 AI（Enterprise RAG / AI Agent）落地應用** 的軟體工程師。

回顧我的職涯軌跡，是一條不斷挑戰更高複雜度與嚴苛場景的演進之路：從貼近實體產線的 **智慧製造系統（MES / ERP / WMS）** 起步，接著投入高併發流量的 **17Life（17APP）電商平台**；隨後進入對資安與帳務正確性要求極高的金融領域，歷練 **合作金庫特店收單系統** 與 **KGI 凱基銀行新一代網路銀行系統**；近年則進一步深耕高複雜度臨床醫療場域的 **新光醫院 HIS 系統**，以及結合 Azure 雲端與生成式 AI 的 **AIP 企業級 AI 平台與 RAG 架構實踐**。

---

## 🧭 職涯演進與代表性實戰經歷 (Career Timeline)

### Stage 1｜智慧製造與跨系統整合：MES / ERP / WMS / SCM

職涯第一站深入製造業產線現場，處理實體機台與企業資訊系統間的高頻率資料交換，奠定了紮實的系統分析與資料庫交易設計基本功：

- **🏭 產線製程與工單追蹤管理**：實作 WIP（在製品）全生命週期追蹤、工單派工、現場報工與製程良率統計分析。
- **🔌 機台設備聯網與資料採集**：透過 IoT、PLC、SECS/GEM 與 API 串接產線機台，即時採集設備參數、計算稼動率（OEE）並實作異常告警監控。
- **🔄 企業級跨系統雙向整合**：打通 **MES（製造執行系統）** 與 **ERP（企業資源規劃）**、**WMS（倉儲物流管理）**、**SCM（供應鏈管理）**、**SPC（統計製程品管）** 及條碼與產線設備資料串接，確保跨系統庫存與工單帳務一致。
- **📊 工廠戰情室與即時看板**：建置現場端電子看板（Dashboard）、即時產能分析與自動化排程報表。

---

### Stage 2｜高併發電商平台與 APP 後端：17Life（17APP）

帶著對複雜商業流程的掌握，我轉戰面向廣大消費者的網際網路電商場景，迎接大流量與高頻率讀寫的挑戰：

- **🛒 17Life 電商 APP 與平台架構升級**：參與電商 APP 後端 API、高流量商品促銷、訂單交易與金流／物流模組重構，確保系統在熱門檔期流量洪峰下的穩定回應。
- **⚡ 資料庫與查詢效能調校**：針對高併發讀寫之 `SQL Server` 核心資料表與複雜 `Stored Procedure` 進行執行計畫分析、索引優化與批次處理重構，大幅改善交易吞吐量並降低鎖定瓶頸。

---

### Stage 3｜銀行收單金流與大型全端架構：合作金庫銀行（TCB ABMS）

隨後我跨入金融科技領域，在零帳務容錯的銀行收單系統中擔任全端主力開發，並將自動化測試工程導入研發流程：

- **💳 特店收單業務管理系統（.NET 6 + Angular 13）**：基於 `.NET 6 Web API` 後端與 `Angular 13 (ngx-admin)` 前端框架，負責特店進件申請、審核簽核工作流與特店資料全生命週期管理。
- **🏦 多元支付金流整合與清算報表**：串接 **台灣 Pay（TWPay）**、**悠遊卡（EasyCard）** 等支付通道，負責大批量交易清算、跨日對帳批次作業與各類業務財務報表匯出。
- **🧪 Playwright E2E 自動化測試防線**：針對高風險核心進件與交易流程，建置模組化的 `Playwright` 端到端自動化測試與 AI 測試 Agent 工作流（涵蓋 `Discuss`、`Plan`、`Implement`、`Run`、`Archive`），大幅降低回歸測試成本。

---

### Stage 4｜高資安規格與大型網銀核心：KGI 凱基銀行

在累積豐富的金融業務與金流經驗後，我進一步參與大型商業銀行核心外圍與新一代網路銀行系統的演進：

- **🏛️ 新一代網路銀行後端核心開發**：遵循乾淨分層架構（Clean Architecture），參與網銀核心後端 API 與微服務模組開發，實作高資安規格之身份驗證與金融交易流程控制。
- **🔗 跨系統主機電文串接與高可用性設計**：負責網銀中台與銀行核心主機／周邊系統之電文串接與非同步處理，確保高併發在線查詢與資金交易時的資料一致性與低延遲表現。

---

### Stage 5｜醫療核心系統與企業 AI 平台落地：新光醫院 & AIP 專案

近年我將歷經製造、電商與金融淬鍊的架構能力，投入到高度複雜的醫療資訊系統與最前沿的生成式 AI 企業落地專案：

- **🏥 新光醫院 — 醫療 HIS 核心與外圍系統開發**
  - 參與門診、急診、住院醫令流程、電子病歷與健保申報檢核邏輯開發，並負責醫療業務跨系統整合，在容錯率極低的臨床醫療環境中確保系統的高穩定度與即時回應。
- **🤖 AIP 企業級 AI 平台專案（Generative AI & Enterprise RAG）**
  - **企業知識庫與混合檢索架構**：運用 `Azure Document Intelligence` 進行高精準度的複雜文件解析（表格、PDF、非結構化文件），結合 `Azure AI Search` 打造具備向量與語意重排序（Semantic Reranking）的企業級 RAG 知識庫。
  - **AI Agent 與業務流程自動化**：結合 `Semantic Kernel`、`LangChain`、`Azure Functions` 與 `Logic Apps`，實作具備工具呼叫（Tool Calling）與多步驟協作能力的智慧型 Agent，將生成式 AI 轉化為真正解決業務痛點的生產力引擎。

---

## 🛠️ 四大核心技術支柱 (Core Tech Stack)

在技術選型與工程實踐上，我推崇 **乾淨架構（Clean Architecture）**、**領域驅動思維** 與 **可長期維運的軟體設計**：

| 領域維度 | 核心技術與實戰能力 |
| :--- | :--- |
| **後端核心架構與 API** | `C#`、`.NET 8 / .NET 6 / .NET Core`、`ASP.NET Web API`、`Entity Framework Core`、`Dapper`、`Clean Architecture`、微服務／分散式系統與跨系統電文／API 整合 |
| **資料庫與高併發調校** | `SQL Server (T-SQL)`、大型資料庫 Schema 設計、複雜 `Stored Procedure` 效能調校、大量帳務／產線資料批次處理（Batch Processing）與交易鎖定最佳化 |
| **全端開發與自動化測試** | `Angular`、`TypeScript`、`RxJS`、`ngx-admin`、`Playwright (E2E Automated Testing)`、`Docker`、`Git`、`CI/CD Pipeline` |
| **Azure 雲端與 AI 應用** | `Azure AI Search`、`Azure Document Intelligence`、`Azure Functions`、`Azure Logic Apps`、`Enterprise RAG`、`Semantic Kernel`、`LangChain`、`AI Agent Skills` |

---

## 💡 工作與技術哲學 (Engineering Philosophy)

1. **釐清脈絡，聚焦核心**：無論面對產線機台訊號、電商高併發訂單、銀行收單帳務還是醫療臨床醫令，動手寫 Code 前必先梳理核心資料流與系統邊界，讓架構自然貼合業務本質。
2. **風險導向，品質對齊**：專案時程緊湊時，優先鎖定高風險關鍵路徑（如身份驗證、金流清算、跨系統拋轉），與 PM、SA、QA 團隊高度對齊驗收標準與自動化測試範圍，確保品質與交付時程兼顧。
3. **實戰落地，人機協作**：主張技術必須服務於真實業務場景，拒絕「為了玩工具而玩工具」；同時善用 AI Agent 加速架構探索與測試驗證，轉化為團隊可持續維護的工程資產。

---

## ✍️ 關於這個 Blog (Why This Blog)

這裡記錄了我在企業級系統開發、資料庫調校、雲端架構與 AI 落地應用上的實戰筆記與踩坑心得。我希望這個部落格能同時成為：

1. 📚 **個人的實戰知識庫**：沉澱高價值的架構決策脈絡與問題排查過程。
2. 🪟 **專業與職涯的窗口**：讓合作夥伴、技術同好與獵頭顧問能快速了解我的技術廣度、實戰深度與工程思維。
3. 💡 **開發者社群的共好分享**：為同樣深耕 `.NET`、`Azure`、企業系統整合與 `Generative AI` 的工程師提供可直接參考的實戰指南。

---

📬 **歡迎交流與合作**  
如果你對我的文章內容有任何疑問，或有技術交流、職涯切磋與專案合作需求，歡迎透過以下社群連結與我聯繫：

- 💬 **Discord 技術社群**：[點選我](https://discord.com/invite/7z9QcbAP5X)
- 💼 **LinkedIn**：[點選我](https://www.linkedin.com/in/%E5%9C%8B%E5%8B%9B-%E9%A6%AC-760860201/)
- 🐙 **GitHub**：[點選我](https://github.com/JontCont)
- 📸 **Instagram**：[點選我](https://www.instagram.com/johncons_rd/)
- 🧵 **Threads**：[點選我](https://www.threads.com/@johncons_rd)
- 📘 **Facebook**：[點選我](https://www.facebook.com/john.conte.749662/)
