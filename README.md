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


**車輛與帳務管理系統架構圖** <BR>
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

