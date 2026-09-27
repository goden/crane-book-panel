# crane-book-panel

## 簡介

建立一套專為吊車公司量身打造的 IT 系統，需要兼顧傳統重機具產業的派班邏輯與現代化的軟體架構。
(Establish a tailored IT ecosystem for the crane industry that harmonizes legacy heavy machinery operations with cutting-edge software engineering.)

1. 系統核心模組規劃
吊車業的營運核心在於「人、車、排程、帳務」的精準媒合。

    | 模組名稱 | 核心功能 | 關鍵資料與狀態 |
    | --- | --- | --- |
    | 機具與車輛管理 | 吊車規格（噸數、臂長）、保養紀錄、驗車排程 | 車輛狀態（可用、維修中、出勤中）、檢驗到期日 |
    | 派車與排程系統 | 視覺化甘特圖排班、臨時調度、司機人員與操作手媒合 | 任務時間、地點、指派車輛、指派 |
    | 報價與帳務管理 | 出車單電子化、報價單生成、應收帳款（月結/現金）| 趟次計費標準、超時費用計算、請款單狀態 |
    | 人員與證照管理 | 司機/助手出勤紀錄、特殊機具操作證照效期追蹤 | 證照到期預警、當月出勤工時 |

2. Python 核心技術選型
為確保系統的擴充性與開發效率，建議採用前後端分離架構，並導入現代化的 Python 生態系
- 後端框架：FastAPI
    具備極高的效能與非同步處理能力，且內建自動化 OpenAPI (Swagger) 文件生成，能大幅減少 API 文件的維護成本。
- 資料庫與 ORM：PostgreSQL + SQLAlchemy
    關聯式資料庫適合處理複雜的排程與帳務邏輯。透過 SQLAlchemy 的 ORM 映射，能以物件導向的方式操作資料，這與處理企業級系統（如 Spring/Oracle 環境）的邏輯高度一致。
- 前端介面：Angular 或 Vue.js
    若團隊已有熟悉的 SPA 框架（如 Angular），可直接介接 FastAPI 提供的 RESTful API；若需快速建立內部後台，也可評估使用純 Python 的 UI 框架如 NiceGUI 或 Streamlit 進行雛形開發。


## **車輛與帳務管理系統架構圖** <BR>

前後端分離、非同步任務處理與多層快取之分層架構 (懸停節點查看連線與說明)

 ```mermaid
graph LR;
    frontend["`前端介面 <BR> _Angular/WebUI_`"];
    gateway["`API閘道器 <BR> _Nginx/Reserved Proxy_`"]
    backend["`後端服務 <BR> _FastAPI(Core)_`"]
    cache["`快取服務 <br> _Redis Cache_`"]
    database["`關聯式資料庫 <br> _PostgreSQL_`"]
    queue["`任務佇列 <br> _Celery Worker_`"]
    frontend --HTTPS--> gateway --PROXY--> backend;
    backend -.AMQP.-> cache;
    backend -.asyncpg.-> database;
    backend -.IN-Memory.-> queue;
```

| 角色 | 用途                                                                                                                  |
| --- |-----------------------------------------------------------------------------------------------------------------------|
| 前端介面 | 使用者操作介面，負責車輛監控圖表、帳務報表顯示及即時派單操作。透過 REST API 與 WEB Socket 與後端服務同步資料。        |
 | API 閘道 | 統一系統入口點，處理 SSL/TLS 憑證解密、請求速率限制 (Rate Limiting)。靜態檔案快取及 API 請求的反向代理入口。          |
| 後端服務 | 採用 Python FastAPI 建立的高效能非同步 API 服務，負責車輛狀態計算、帳務試算、權限控管，並協調快取、資料庫與背景任務。 | 
| 快取服務 | 儲存高頻存取的車輛即時 GPS 定位，使用者 Session 與 API 回應快取，大幅降低主資料庫讀取壓力與提升回應速度。             |
| 關聯式資料庫 | 主要資料庫，負責 ACID 事務保障，儲存車輛主檔、駕駛資料、交易紀錄及歷史帳務等核心結構化資料。                          |
| 任務佇列 | 背景任務處理器，搭配 RabiitMQ/Redis 訊息仲介，負責費時的月度對帳單生成、批次車輛數據匯出與定時排程通知發送。          |


3. 軟體開發生命週期 (SDLC) 執行計畫

利用熟悉的 VS Code 結合自定義 AI Agent 工作流，能大幅加速以下各階段的基礎程式碼生成與測試腳本編寫。

3.1. 需求分析與系統設計 (System Analysis)

繪製領域模型 (Domain Model) 並確定資料庫 Schema。將車輛噸數、出車計費邏輯（包月、包日、單趟）等產業 Know-how 轉化為資料表關聯。

3.2. 開發實作 (Implementation)

透過 FastAPI 實作各領域驅動的 API 端點。建議優先完成「車輛建檔」與「派車單建立」兩大核心，讓使用者能盡早測試排班邏輯是否符合實務需求。

3.3. 自動化測試建置 (Testing)

在此階段導入 Pytest 進行單元與整合測試。針對 E2E (端到端) 的 UI 測試，可以採用 Playwright (Python 版) 搭配 Allure Report，產出精美的視覺化測試報告，這套測試架構能無縫銜接至 CI/CD 流程中。

3.4. 部署與維運 (Deployment & Maintenance)

使用 Docker 進行容器化，確保開發、測試與正式環境的一致性。初期可部署於 AWS EC2 或 GCP Compute Engine，並透過 GitHub Actions 建立自動化部署管線。

4. 文件備存與管理策略

系統文件的妥善保存是專案成功的關鍵，建議採用 Docs-as-Code (文件即程式碼) 的策略：

4.1. 架構與流程文件：MkDocs (Material Theme)

使用 Markdown 撰寫系統架構、業務邏輯與操作手冊。MkDocs 是以 Python 為基礎的靜態網站產生器，能與原始碼放在同一個 Git 儲存庫中進行版本控制，且介面極具專業感。

4.2. API 規格文件：FastAPI 內建 Swagger UI

只要程式碼中的 Pydantic Model 與 Type Hints 定義清楚，系統會自動生成並更新 API 規格與測試介面，完全免除手動維護 API 文件的負擔。

4.3. 環境與依賴管理：Poetry

捨棄傳統的 `requirements.txt`，改用 Python 現代化的套件管理工具 Poetry (`pyproject.toml`)，明確鎖定所有依賴套件的版本，確保未來接手的開發者能一鍵還原開發環境。

5. PostgreSQL 資料庫 Schema 規劃

針對吊車公司的營運模式，資料庫設計的核心在於處理「客戶叫車」、「車輛與司機調度」以及「後續帳務」這三個主要環節。使用 PostgreSQL，我們可以利用其關聯式特性，確保資料的一致性與完整性。

<img src="https://encrypted-tbn1.gstatic.com/licensed-image?q=tbn:ANd9GcTlqDcYycMFAZ7X1EsGHQyQ6D4cGE0ojgw8Hfc-hl6CgUG0PndNgJoZ8dw9xiZQqe27WDEgJ2RSR5bMZ_M" width="50%">

施工現場吊車作業. 來源： Andrey Atanov / Getty Images<br>

<img src="https://encrypted-tbn2.gstatic.com/licensed-image?q=tbn:ANd9GcRZne9teqeFwzLsG8pxErbOS8lNPcteNPKeTOo_mIWfKDab-Rq55YkqDyLDiNLU7TUZYtp2pFmedqZKa7M" width="50%">

資料塑模概念. 來源： zuamir / Getty Images<br>

<img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRPAGF2H3pewCAubfKWonhpe3-8_qebaVql0IBYBiPaP_lKwLFESboswzA&s=10" width="50%">

派車系統 ER 圖概念. 來源： Latest Projects on Java, JSP, Python<br>

<img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTkudghynkYCZMhaI_wHD1kACzdlICn-Jh8OqBZfFqsgJmZgHSHFU8rsqE&s=10" width="50%">

PostgreSQL 架構. 來源： Medium / Architecture of PostgreSQL DB. Basic architecture of Database<br>

5.1. 核心資料表 (Schema) 規劃

以下是針對吊車系統核心業務所設計的三大資料表關聯架構：客戶 (Customers)、車輛 (Vehicles) 與派車單 (Dispatch Orders)。

- **客戶資料表 (customers)**<BR>儲存叫車的企業或個人資訊，是產生報價單與請款單的基礎。

    | 欄位名稱 (Column) | 資料型態 (Type) |屬性 (Constraints)|說明 (Description)
    |-------------------|-----------------|---|---|
    | customer_id       | UUID / SERIAL   | PRIMARY KEY | 客戶唯一識別碼 |
    | company_name      | VARCHAR(100)    | NOT NULL | 公司名稱或聯絡人姓名 |
    | tax_id            | VARCHAR(8)      | UNIQUE | 統一編號（若為企業） |
    | contact_person    | VARCHAR(50)     | NOT NULL | 主要聯絡人 |
    | phone_number      | VARCHAR(20)     | NOT NULL | 聯絡電話 |
    | billing_address   | VARCHAR(255)    |  | 帳單寄送地址 |
    | payment_terms     | VARCHAR(50)     | DEFAULT '現金' | 付款條件（如：現金、月結30天）|
    | created_at        | TIMESTAMP       | DEFAULT NOW()   | 建立時間        |

- **車輛與機具資料表 (vehicles)**<BR>管理公司內部所有吊車、卡車或特殊機具的狀態與規格。

    | 欄位名稱 (Column)  | 資料型態 (Type) | 屬性 (Constraints)  | 說明 (Description)                         |
    |--------------------|-----------------|---------------------|--------------------------------------------|
    | vehicle_id         | UUID / SERIAL   | PRIMARY KEY         | 車輛唯一識別碼                             |
    | license_plate      | VARCHAR(20)     | UNIQUE, NOT NULL    | 車牌號碼                                   |
    | vehicle_type       | VARCHAR(50)     | NOT NULL            | 車輛種類（如：25噸螃蟹車、45噸全距車）     |
    | tonnage            | NUMERIC(5,2)    | NOT NULL            | 吊車噸數                                   |
    | status | VARCHAR(20)     | DEFAULT 'AVAILABLE' | 狀態（AVAILABLE, MAINTENANCE, DISPATCHED） |
    | next_inspection | DATE            |                     | 下次驗車/保養日期                          |


- **派車單資料表 (dispatch_orders)**<BR>這是系統的核心交易表，負責將「客戶」、「車輛」與「時間地點」連結起來。

    | 欄位名稱 (Column)      | 資料型態 (Type) | 屬性 (Constraints) | 說明 (Description) |
    |------------------------|-----------------| --- | --- |
    | order_id               | UUID / SERIAL   | PRIMARY KEY | 派車單唯一識別碼 |
    | customer_id            | UUID / INT      | FOREIGN KEY | 關聯至 customers.customer_id | 
    | vehicle_id             | UUID / INT      | FOREIGN KEY | 關聯至 vehicles.vehicle_id |
    | driver_id              | UUID / INT      | FOREIGN KEY | 關聯至司機/操作手資料表（暫略）|
    | job_date               | DATE            | NOT NULL | 預定施工日期 |
    | start_time             | TIME            | NOT NULL | 預計抵達/開始時間 |
    | end_time               | TIME            |  | 實際結束時間 |
    | job_location           | VARCHAR(255)    | NOT NULL | 施工地點 |
    | job_description        | TEXT            |  | 施作內容描述（如：吊掛鋼樑） |
    | price_type | VARCHAR(20)     | NOT NULL | 計費方式（包月、包日、半日、單趟） |
    | quoted_price | NUMERIC(10,2) | NOT NULL | 報價金額 |
    | order_status | VARCHAR(20) | DEFAULT 'PENDING' | 狀態（PENDING, IN_PROGRESS, COMPLETED, CANCELED）|


