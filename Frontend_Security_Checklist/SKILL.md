Name: frontend-security-checklist
Make by Kingsley
GitHub: @KingsleyHK0323
github.com/KingsleyHK0323



\### 1. 輸入驗證、輸出編碼與注入防禦（Input, Encoding \& Injection）



\* \[ ] \*\*DOM XSS 防禦\*\*：禁止以不受信任的資料操作 `innerHTML`、`outerHTML`、`document.write()`、`insertAdjacentHTML()`、`eval()`、`new Function()`，以及 `setTimeout()` / `setInterval()` 傳入字串。

\* \[ ] \*\*框架渲染與富文本淨化\*\*：避免使用 `dangerouslySetInnerHTML`（React）、`v-html`（Vue）、`\\\[innerHTML]`（Angular bypassSecurityTrust）；富文本渲染前須以 \*\*DOMPurify\*\* 設定嚴格白名單（允許的 Tag 與 Attribute）進行淨化。

\* \[ ] \*\*URL 與導向安全（Open Redirect \& Scheme Injection）\*\*：驗證動態 `href`、`src`、`action` 與 `window.location` 導向目標，阻擋 `javascript:`、`data:`、`vbscript:` 協議；站內跳轉僅允許相對路徑或經過白名單比對的網域。

\* \[ ] \*\*原型污染（Prototype Pollution）防禦\*\*：合併物件或以動態 Key 賦值（如 `obj\\\[key] = val`）時，過濾 `\\\_\\\_proto\\\_\\\_`、`constructor` 與 `prototype`；優先使用 `Object.create(null)` 或 `Map`，並對 Schema 進行驗證（如 Zod / Yup）。

\* \[ ] \*\*表單與檔案上傳前端預檢\*\*：前端可先檢查檔案副檔名、MIME Type 與檔案大小提升體驗，但\*\*嚴禁僅靠前端檢查\*\*作為安全防線（後端必須重新驗證內容與魔術位元組 Magic Bytes）。



\### 2. 身份驗證、工作階段與存取控制（Auth, Session \& Access Control）



\* \[ ] \*\*Token 儲存策略\*\*：避免將高權限 Access Token / Refresh Token 放進 `localStorage` 或 `sessionStorage`；優先採用 `HttpOnly; Secure; SameSite` Cookie 管理 Session，或搭配短期 Access Token（記憶體暫存）與安全 Refresh 機制。

\* \[ ] \*\*CSRF 防禦\*\*：使用 Cookie 驗證時設定 `SameSite=Lax` 或 `Strict`（跨站嵌入式情境需明確評估 `SameSite=None; Secure`），並對關鍵狀態變更操作實施 Anti-CSRF Token 或 Custom Header 驗證。

\* \[ ] \*\*前端路由與 UI 權限控管\*\*：未登入或權限不足者應阻擋存取受限路由並導向登入頁；按鈕/選單隱藏僅屬 UX 層級，所有資料與操作皆須由後端 API 強制執行授權（Authorization）。

\* \[ ] \*\*登出與工作階段清理\*\*：登出時需清除記憶體狀態、前端快取、相關 Storage/IndexedDB 資料，並呼叫後端撤銷 Token / Session。

\* \[ ] \*\*敏感操作二次確認與重驗證\*\*：變更密碼、綁定裝置、大額交易或刪除帳號等高風險操作，應要求重新驗證密碼/OTP 或二次確認。



\### 3. HTTP 安全標頭與傳輸層安全（HTTP Headers \& Transport Security）



\* \[ ] \*\*內容安全政策（CSP - Content Security Policy）\*\*：明確定義 `default-src 'self'`、`script-src`、`style-src`、`img-src`、`connect-src`、`font-src`、`frame-ancestors`；避免 `unsafe-inline` 與 `unsafe-eval`，必要時改用 `nonce` 或雜湊值（Hash）。

\* \[ ] \*\*反點擊劫持（Clickjacking）\*\*：設定 `X-Frame-Options: DENY` 或 `SAMEORIGIN`，並以 CSP `frame-ancestors` 精準限制可嵌入的父網域。

\* \[ ] \*\*MIME 嗅探與強制 HTTPS\*\*：設定 `X-Content-Type-Options: nosniff`；全站啟用 HTTPS 並設定 `Strict-Transport-Security`（HSTS，建議包含 `includeSubDomains` 與適當 `max-age`）。

\* \[ ] \*\*跨來源隔離與來源標頭（COOP / COEP / CORP / Referrer-Policy）\*\*：視需要使用 `Cross-Origin-Opener-Policy`（COOP）、`Cross-Origin-Embedder-Policy`（COEP）、`Cross-Origin-Resource-Policy`（CORP）；設定 `Referrer-Policy`（如 `strict-origin-when-cross-origin`）避免敏感查詢參數隨 Referer 外洩。

\* \[ ] \*\*CORS 精準設定\*\*：後端 `Access-Control-Allow-Origin` 不可使用 `\\\*` 搭配 `Access-Control-Allow-Credentials: true`，應逐一核對允許來源與方法。



\### 4. 供應鏈、第三方套件與建置安全（Supply Chain \& Build Pipeline）



\* \[ ] \*\*相依性漏洞管理\*\*：鎖定 Lockfile（`package-lock.json`、`pnpm-lock.yaml`、`yarn.lock`），於 CI 執行 `npm audit` / `pnpm audit` 或整合 Dependabot / Renovate / Snyk，並建立高危漏洞（Critical/High）修補 SLA。

\* \[ ] \*\*套件來源與惡意腳本防範\*\*：審查新引入套件的維護狀態、下載量與授權；安裝套件時留意 `postinstall` 腳本風險，必要時搭配 `--ignore-scripts` 並審核。

\* \[ ] \*\*子資源完整性（SRI - Subresource Integrity）\*\*：引用第三方 CDN 腳本或樣式時加上 `integrity="sha384-..."` 與 `crossorigin="anonymous"`，並備有降級或本地託管方案。

\* \[ ] \*\*第三方腳本（Analytics / Tag Manager / Chat Widget）控管\*\*：評估第三方 SDK 權限與資料流向，避免直接將不受控的第三方 Tag Manager 注入高權限頁面；監控第三方腳本效能與行為異常。

\* \[ ] \*\*建置產物與機密防洩漏\*\*：確認程式碼、環境變數（`VITE\\\_\\\*`、`NEXT\\\_PUBLIC\\\_\\\*` 等公開前綴）與 Git 歷史中無 API Secret Key、私鑰、內部網域或測試帳密；正式環境停用或妥善保護 Source Maps（`.map`）。



\### 5. 客戶端儲存、快取與離線安全（Client Storage \& Offline / PWA）



\* \[ ] \*\*敏感資料不落地\*\*：不在 `localStorage`、`sessionStorage`、IndexedDB 或 Cache Storage 明文存放密碼、個人身分證號、付款卡號、完整 Token 或機密業務資料。

\* \[ ] \*\*快取控管（Cache-Control）\*\*：針對個人化資料、帳單、帳號設定等敏感回應，設定 `Cache-Control: no-store`（或 `private, no-cache`），避免共用電腦或瀏覽器快取殘留。

\* \[ ] \*\*Service Worker 與 PWA 快取安全\*\*：檢查 Service Worker 攔截請求的範圍與快取策略，避免將敏感 API 回應快取到離線快取中，並確保 SW 腳本本身經 HTTPS 與嚴格快取更新機制控管。



\### 6. 跨視窗通訊、WebSocket 與即時資料安全（Messaging \& Realtime）



\* \[ ] \*\*`window.postMessage` 安全驗證\*\*：接收訊息的監聽器必須嚴格驗證 `event.origin`（使用精準白名單而非字串包含判斷），並驗證 `event.source` 與資料結構。

\* \[ ] \*\*外部連結分頁釣魚防護\*\*：所有 `target="\\\_blank"` 連結加上 `rel="noopener noreferrer"`（現代瀏覽器雖有預設保護，顯式宣告仍為最佳實踐）。

\* \[ ] \*\*WebSocket / SSE / WebRTC 安全\*\*：使用加密通道（`wss://`、`https://`）；建立連線時驗證 Origin 與身份憑證，並對即時推送的訊息做 Schema 驗證與狀態防重放/防競態處理。



\### 7. 瀏覽器權限、多媒體與裝置 API（Browser APIs \& Permissions）



\* \[ ] \*\*最小權限原則與使用者意圖\*\*：存取相機、麥克風、地理位置、剪貼簿、通知或檔案系統前，必須由明確的使用者互動（如點擊按鈕）觸發，並清楚說明用途。

\* \[ ] \*\*Permissions-Policy 標頭\*\*：透過 HTTP `Permissions-Policy` 停用網站不需要的瀏覽器功能（如 `camera=(), microphone=(), geolocation=()`），降低被嵌入或 XSS 濫用裝置 API 的風險。

\* \[ ] \*\*剪貼簿與 Canvas / WebGL 資安\*\*：寫入剪貼簿前避免夾帶隱藏惡意指令；使用 HTML5 Canvas 或 WebGL 處理跨來源影像時留意 Tainted Canvas 限制與記憶體釋放。



\### 8. 前端邏輯、防濫用與業務韌性（Logic, Abuse \& Resilience）



\* \[ ] \*\*前端驗證僅作 UX，後端必做校驗\*\*：必填、格式、長度、數值範圍、價格與狀態流轉皆需在後端再次驗證，防止透過 DevTools 或 API 請求竄改。

\* \[ ] \*\*防重送與頻率限制配合\*\*：表單提交、付款、發送驗證碼、抽獎或按讚等操作，前端應加上 Loading 鎖定 / Debounce / Idempotency Key 提示，後端同步實施 Rate Limiting。

\* \[ ] \*\*正則表達式與遞迴安全（ReDoS \& DoS）\*\*：避免在前端處理超大字串時使用易發生回溯爆炸（Backtracking）的複雜正則表達式，或無上限的遞迴渲染。



\### 9. 隱私權、Cookie 同意與合規性（Privacy \& Compliance）



\* \[ ] \*\*同意管理平台（CMP - Consent Management Platform）\*\*：依目標市場法規（如 GDPR、CCPA/CPRA 或在地個資規範），在取得使用者同意前，不載入非必要的追蹤 Cookie 與分析/廣告腳本。

\* \[ ] \*\*追蹤與日誌去敏化\*\*：前端埋點（Telemetry / Analytics）與錯誤回報事件中，過濾 Email、電話、Token、Authorization Header、密碼與查詢字串中的敏感參數。



\### 10. 錯誤處理、可觀測性與上線維運（Error Handling, Observability \& Ops）



\* \[ ] \*\*全域錯誤邊界與例外捕捉\*\*：設定 Error Boundary（如 React Error Boundary）與 `window.onerror` / `unhandledrejection`，避免未捕捉例外導致整頁白屏，同時不向使用者顯示內部路徑或堆疊資訊。

\* \[ ] \*\*CSP 違規回報（CSP Reporting）\*\*：設定 `report-uri` / `report-to` 蒐集真實使用者的 CSP 違規紀錄，及早發現 XSS 嘗試、被劫持的第三方腳本或誤擋的合法資源。

\* \[ ] \*\*發布與回滾機制\*\*：建立自動化安全掃描（SAST / 依賴檢查）與快速回滾（Rollback）流程，一旦發現重大前端漏洞可於數分鐘內撤回或發布修補版本。



