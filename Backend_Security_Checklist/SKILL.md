Name: backend-security-checklist
Make by Kingsley
GitHub: @KingsleyHK0323
github.com/KingsleyHK0323



\### 1. 輸入驗證、注入防禦與資料處理（Input Validation \& Injection）



\* \[ ] \*\*參數化查詢（Parameterized Queries / ORM）\*\*：所有資料庫存取皆使用 Prepared Statement 或 ORM 參數綁定，嚴禁以字串拼接組合 SQL / NoSQL 查詢，防範 SQL Injection（SQLi）。

\* \[ ] \*\*命令與系統呼叫安全（Command Injection）\*\*：避免在後端執行外部系統指令（如 `exec`、`spawn`）；若無可避免，應使用陣列傳參而非 Shell 字串拼接，並對參數做嚴格白名單校驗。

\* \[ ] \*\*路徑穿越防禦（Path Traversal）\*\*：處理檔案讀取、下載或刪除時，解析並比對正規化後的絕對路徑（如 `path.resolve` / `realpath`），確保嚴格落在允許的根目錄內，阻擋 `../` 或符號連結（Symlink）逃逸。

\* \[ ] \*\*伺服器端請求偽造（SSRF - Server-Side Request Forgery）\*\*：後端代抓外部 URL、Webhook 或圖片時，驗證目標 Protocol（僅限 `http`/`https`）、網域/IP 白名單，並封鎖內網 IP 段（如 `127.0.0.1`、`169.254.169.254`、RFC1918 私有網段）與重新導向跳轉。

\* \[ ] \*\*結構化 Schema 驗證與型別檢查\*\*：對 Request Body、Query、Path Parameters 與 Headers 進行嚴格 Schema 驗證（如 Zod、Joi、Pydantic），拒絕非預期欄位、型別錯誤、負數或溢位數值。

\* \[ ] \*\*反序列化安全（Insecure Deserialization）\*\*：避免對不受信任的來源使用原生反序列化機制（如 Pickle、Java Native Serialization、PHP `unserialize`）；優先使用安全的資料交換格式（如 JSON）並驗證結構。

\* \[ ] \*\*XML / XXE 防禦\*\*：解析 XML 時停用外部實體（XXE - XML External Entity）與 DTD 載入。



\### 2. 身份驗證與工作階段管理（Authentication \& Session Management）



\* \[ ] \*\*密碼雜湊演算法\*\*：密碼儲存嚴禁明文或使用 MD5 / SHA-1 / SHA-256 等快速雜湊；必須使用慢速加鹽雜湊演算法，如 \*\*Argon2id\*\*、\*\*bcrypt\*\* 或 \*\*PBKDF2\*\*（搭配適當成本參數 Cost Factor）。

\* \[ ] \*\*暴力破解與憑證填充防護\*\*：登入、忘密與 OTP 驗證端點應實施 Rate Limiting、帳號鎖定或漸進式延遲（Exponential Backoff），並避免回傳「帳號存在與否」的明確差異訊息（防列舉 Enumerable Username）。

\* \[ ] \*\*Token 與 Session 安全性\*\*：

\* Session ID 需具足夠亂數熵（CSPRNG），登入成功後重新產生（防 Session Fixation），登出時於伺服器端確實失效。

\* 若使用 JWT，嚴格驗證演算法（禁止 `alg: none` 與演算法混淆）、簽章金鑰強度、過期時間（`exp`）、發行者（`iss`）與受眾（`aud`），並規劃撤銷/黑名單機制。





\* \[ ] \*\*密碼重設與 MFA 設計\*\*：重設密碼連結/Token 應具備高亂數性、短效期與單次使用限制；多因子驗證（MFA / OTP）應限制嘗試次數並防範重放攻擊。



\### 3. 存取控制與授權防禦（Access Control \& Authorization）



\* \[ ] \*\*防範不安全直接物件參考（IDOR / BOLA）\*\*：每次存取或修改特定資源（如 `/api/orders/{id}`、`/api/files/{id}`）時，皆須比對當前登入者的身份與該資源的擁有權/所屬組織，不可僅依賴前端傳來的 ID 或路由存在性。

\* \[ ] \*\*功能與管理端點權限控管\*\*：管理後台（Admin）、內部 API 與敏感操作路由，必須在中介層（Middleware）逐一驗證角色/權限（RBAC / ABAC），預設拒絕（Default Deny）。

\* \[ ] \*\*防止水平與垂直越權\*\*：確認一般使用者無法存取他人資料（水平越權），也無法透過修改角色欄位、請求參數或直接呼叫 API 提升為管理員（垂直越權）。



\### 4. API 安全、流量控管與防濫用（API Security \& Rate Limiting）



\* \[ ] \*\*速率限制與配額管理（Rate Limiting \& Quotas）\*\*：對公開 API、登入、簡訊/Email 發送、搜尋與重運算端點設定依 IP、帳號或 API Key 的請求頻率限制，防範暴力破解、爬蟲與應用層 DoS。

\* \[ ] \*\*分頁與查詢上限（Pagination \& Payload Limits）\*\*：列表查詢強制設定預設與最大 `limit`（如最多 100 筆），並限制 Request Body 最大大小與 JSON 巢狀深度，防範資源耗盡（DoS）。

\* \[ ] \*\*CORS 與跨來源策略\*\*：精準設定 `Access-Control-Allow-Origin` 白名單，避免使用 `\*` 搭配 `Access-Control-Allow-Credentials: true`；確認僅開放必要的 HTTP Methods 與 Headers。

\* \[ ] \*\*GraphQL / RPC 特定防護（若適用）\*\*：限制查詢深度（Query Depth）、查詢複雜度（Query Complexity）與批次查詢數量，正式環境停用 Introspection（視需求）。



\### 5. 機密管理、加密與金鑰生命週期（Secrets, Crypto \& Key Management）



\* \[ ] \*\*嚴禁硬編碼機密（No Hardcoded Secrets）\*\*：API Key、資料庫密碼、JWT Secret、私鑰與雲端憑證不得寫入程式碼或提交至 Git 儲存庫；使用 Secrets Manager 或環境變數注入，並搭配 Secret Scanning 工具檢查。

\* \[ ] \*\*傳輸中加密（Encryption in Transit）\*\*：全站強制使用 HTTPS（TLS 1.2 以上，建議 TLS 1.3），停用弱加密套件與過時協定（SSLv3 / TLS 1.0 / TLS 1.1）。

\* \[ ] \*\*靜態加密（Encryption at Rest）\*\*：敏感個人資料（PII）、醫療/財務紀錄或憑證在資料庫與檔案儲存層應視合規需求進行加密，並妥善管理加密金鑰（KMS）與金鑰輪替（Key Rotation）流程。



\### 6. 檔案上傳與外部整合安全（File Upload \& Third-Party Integration）



\* \[ ] \*\*檔案上傳安全檢核\*\*：

\* 驗證檔案大小、副檔名白名單與真實檔案內容（Magic Bytes / 圖片重新編碼）。

\* 重新產生隨機檔名儲存，避免使用原始檔名覆寫系統檔案或執行惡意腳本。

\* 上傳檔案建議存放於獨立網域或物件儲存（如 S3），並停用執行權限與設定適當的 `Content-Disposition` / `X-Content-Type-Options: nosniff`。





\* \[ ] \*\*Webhook 與第三方回呼驗證\*\*：接收 Stripe、GitHub、Line 等第三方 Webhook 時，必須驗證簽章（HMAC Signature）與時間戳記（防 Replay Attack），並在非同步佇列中處理重試與冪等性（Idempotency）。



\### 7. 錯誤處理、日誌紀錄與可觀測性（Error Handling, Logging \& Monitoring）



\* \[ ] \*\*安全錯誤回應\*\*：正式環境捕捉未預期例外後，僅回傳通用錯誤訊息與追蹤編號（Request ID / Trace ID），不洩漏 SQL 語法、堆疊追蹤（Stack Trace）、檔案路徑或框架版本。

\* \[ ] \*\*稽核日誌與敏感資料遮罩（Log Sanitization）\*\*：

\* 記錄登入成功/失敗、權限變更、關鍵資料異動與異常存取等稽核事件。

\* 日誌輸出前\*\*嚴格遮罩/排除\*\*密碼、Token、Session ID、信用卡號、身分證字號與完整 Request/Response 機密標頭，並防範日誌注入（Log Injection / CRLF）。





\* \[ ] \*\*安全告警與監控\*\*：監控異常 401/403/429 飆升、資料庫慢查詢或錯誤率突增，建立即時告警機制。



\### 8. 資料庫與基礎設施安全（Database \& Infrastructure）



\* \[ ] \*\*資料庫最小權限\*\*：應用程式連線帳號僅授予所需資料表的 `SELECT`、`INSERT`、`UPDATE`、`DELETE` 等必要權限，禁止使用 `root` / 最高管理員帳號連線，並避免不必要的 DDL 權限。

\* \[ ] \*\*網路隔離與防火牆\*\*：資料庫、快取（Redis）、訊息佇列（RabbitMQ/Kafka）與內部微服務不對公網暴露，僅允許受信任的 VPC / 內網與特定 Port 存取。

\* \[ ] \*\*容器與執行環境安全\*\*：基礎映像檔（Base Image）使用精簡且受信任來源，以非 root 使用者（Non-root User）執行應用程式，並定期掃描 OS 與套件漏洞。



\### 9. 交易一致性、併發與業務邏輯防禦（Business Logic \& Concurrency）



\* \[ ] \*\*冪等性設計（Idempotency）\*\*：付款扣款、建立訂單、發放點數/優惠券等關鍵寫入操作，使用 Idempotency Key 或唯一約束防範網路重試或連點造成的重複執行。

\* \[ ] \*\*Race Condition 與交易隔離\*\*：涉及餘額扣減、庫存鎖定或名額搶購時，使用資料庫 Transaction、悲觀鎖（`SELECT ... FOR UPDATE`）或樂觀鎖（Version / CAS），防止併發超賣或負餘額。

\* \[ ] \*\*業務規則完整校驗\*\*：商品價格、折扣計算、狀態機流轉與數量限制皆由後端依權威資料計算與驗證，不可信任前端傳回的計算結果。



\### 10. 依賴套件、CI/CD 與上線維運（Supply Chain, CI/CD \& Ops）



\* \[ ] \*\*相依性漏洞掃描\*\*：於 CI 流程整合 `npm audit` / `pip-audit` / `cargo audit` / Trivy / Dependabot，及時修補已知漏洞。

\* \[ ] \*\*安全部署與回滾機制\*\*：限制正式環境直接手動修改與部署；建立自動化測試、資料庫 Migration 備份與快速回滾（Rollback）程序。

\* \[ ] \*\*定期安全測試與演練\*\*：針對核心 API 進行安全單元/整合測試、SAST/DAST 掃描，並依風險等級安排滲透測試（Penetration Test）與資安事件應變演練。



