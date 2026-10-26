最後修改日期：`20261026`

# 東京與野澤溫泉旅行頁面：整體設計規格

## 1. 目的與維護目標

本專案是一個手機優先、單一 HTML 檔案的旅行計畫頁面。根目錄的 `index.html` 是唯一保留的專案 HTML 與 GitHub Pages 發佈入口；開發用測試集中在 `test-only/`，舊摘要與技能集中在 `archive/`。這兩個目錄都不屬於現行網站的執行依賴。若 Pages 以分支根目錄發佈，仍只需更新根目錄的 `index.html` 才會更新網站；不要在測試或封存檔案放入私人資料，因為它們若一併推送也可能被網站公開存取。

頁面定位為旅行當天可直接操作的私人 Travel App。使用者透過浮動底部導覽切換首頁、行程與更多三個 View。已開啟的單檔頁面／本機檔案，其文字行程在斷網或 Leaflet CDN 失敗時仍要能正常渲染；地圖圖磚、遠端圖片與即時天氣需要網路。GitHub Pages 首次載入仍需連線；沒有 Service Worker 或完整離線安裝能力。

工作原則：維持單檔發佈、小範圍修改、保護 `trip-data` 與互動契約、每次變更後執行結構與瀏覽器測試。

每次修改正式頁面時，同步更新 `trip-data.lastModified`（ISO 日期）與本文件第一行的最後修改日期（`YYYYMMDD`）。頁尾在 App 版標題下方，以 `<time>` 顯示「最後修改日期：YYYYMMDD」，三個 View 共用；使用低調次要文字色與正常字重，僅日期數字採裝置內建等寬字體，不加膠囊、圓點或底色。文件第一行僅日期數字使用行內代碼字體，不加圖示或粗體。不可改用開啟頁面的當天日期或瀏覽器提供的檔案時間。驗證器會檢查日期有效且 HTML 與文件一致。

## 2. 專案檔案與責任

| 檔案 | 定位 | 修改原則 |
| --- | --- | --- |
| `index.html` | 現行單檔旅行頁與發佈入口 | 唯一發佈目標；包含樣式、資料 JSON、地圖、提醒與全部互動程式。 |
| `archive/daily-itinerary.md`、`archive/local-model-handoff-prompt.md` | 過時的摘要與交接說明 | 僅供回溯；內容未同步現行頁面，請勿作為修改依據。 |
| `archive/.github/skills/travel-plan-viz/` | 舊旅行頁技能、通用契約與引擎 | 完整封存，VS Code 不再從工作區的 `.github/skills/` 自動發現它；現行頁面及測試不依賴此資料夾。 |
| `test-only/package.json`、`test-only/package-lock.json` | 測試套件與鎖定版依賴 | 只供開發；在 `test-only/` 安裝與執行 npm 指令。 |
| `test-only/validate.cjs` | 現行 App 專用資料與結構驗證器 | 在本資料夾獨立維護，不回頭讀封存的通用驗證器。 |
| `test-only/tests/browser-smoke.test.cjs` | Chrome 瀏覽器情境測試 | 修改互動行為時同步維護；只讀取根目錄正式頁面。 |
| `.gitignore` | 測試依賴排除規則 | 排除 `test-only/node_modules/`；不要提交已安裝依賴。 |

### 與封存通用技能契約的差異

封存的技能是舊旅行頁產生流程的歷史參考，**不是**本工作區持續維護時的規格；本頁為既有三 View App，衝突時以本文件及現行頁面為準：

- 使用三 View 底部導覽，不採通用契約的頁首 sticky 快速導覽；重要提醒放在「更多」。
- 行程與地圖各有一組同步日期 Tab；地圖置於行程 View 最下方。
- 地圖點位只提供 Google 地圖入口，不強制 Apple 地圖連結。
- 有些餐廳留在時間軸但不加入地圖，依資料欄位控制，不依名稱文字辨識。
- 結構驗證不涵蓋三 View 與離線行為；互動須執行瀏覽器測試。

## 3. 資訊架構

頁面固定分成三個 App View：

1. 首頁：依日本時間（`Asia/Tokyo`）顯示出發前、旅途中或旅程後狀態；包含當日／預覽主題、下一站、動態天氣、最近提醒、住宿與時間軸摘要。「下一站」先判斷進行中活動，再找最近未開始活動；跨午夜活動在隔日結束前仍顯示進行中，今日全部結束時不把最後一站誤稱下一站。天氣卡透過 Open-Meteo 顯示目前溫度、體感與今日高低溫，可切換目前位置、東京或野澤；目前位置只能在使用者主動選擇後要求定位權限。Tenki.jp 僅提供詳細預報核對連結。API 失敗時以 `trip.weatherLocations.<地點>.seasonal` 降級。重要提醒、今晚住宿、今日站點三張摘要卡與下方行程摘要項目皆支援點擊或鍵盤觸發，直接切換至對應區塊或當日時間軸／住宿節點。
2. 行程：橫向 Day selector 在捲動後貼齊視窗最頂端；切換日期或進入行程 View 時，水平日期列會自動將目前選中的日期按鈕置中顯示。點選行程日期時立即捲動到所選日期的 Day 日期標題，並讓標題顯示於固定日期列下方。每一天在主題下方的注意事項（`tips`）、跨頁快捷按鈕、交通／購票攻略（`transportGuide`／`ticketGuide`）與分組備案（`alternatives`）統一收合於 `.day-notes-details`（預設隱藏），讓第一個時間軸行程節點可直接呈現在畫面上；使用者點擊按鈕即可展開查看完整攻略。地圖自己的日期列僅更新地圖／行程，不將使用者帶離地圖；從時間軸點擊「地圖 #N」跳至地圖後，地圖標題列會顯示 `#backToSlotBtn`（↑ 返回剛才行程），可一鍵捲回原行程節點並短暫高亮。淺色主題的選中日期使用深色實心底與白字，和未選取的淺灰按鈕形成明顯對比。不另加行程數、首站、住宿與預約摘要，以維持頁面簡潔。去程航班是第一天的第一個節點，回程航班是最後一天的最後一個節點。住宿依 `checkIn`／`checkOut` 對應日期，完整顯示為每個住宿日的最後一個節點。每一天只展示第一張有設定的照片作為視覺重點；互動地圖固定放在每日行程區塊最下方。
3. 更多：頂部提供水平固定膠囊列 `#moreQuickNav`（提醒、行前、VJW、退稅、13 湯、小提醒、免責），可一鍵跳至指定區塊並自動隱藏未啟用的可選項目；下方依序為重要提醒、行前須知、Visit Japan Web 準備清單、日本退稅新制、野澤 13 湯與五日巡湯、全程 Tips 與免責聲明。各卡片標題統一帶主題圖示（`.n`），可選區塊由 JSON 產生；資料不存在時以 `hidden` 隱藏。

底部導覽以窄版浮動 Dock 顯示三個入口。向下閱讀且離開頁首後自動隱藏，向上滑、回到頁首、鍵盤聚焦或切換 View 時立即恢復；距離頁底不超過 `100px` 時也恢復導覽，避免到頁底後無法直接切換 View。行程 View 顯示後必須呼叫地圖的 `invalidateSize()`（地圖不可用時由相容的無作用物件處理）。

首頁第一屏是滿版照片封面，高度為可視區扣除底部導覽。封面圖片由 `trip.coverPhoto` 提供，主標、副標與照片地點分別使用 `trip.coverTitle`、`trip.coverSubtitle`、`trip.coverLocation`，避免完整行程標題在手機上產生不自然斷行。圖片使用 `object-fit: cover`，並加深色遮罩確保日期、當日主題、下一站與操作按鈕清楚可讀。首頁隱藏一般頂部列；切換到行程或更多 View 時恢復頂部列。封面以下保留摘要卡與簡短時間軸，向下捲動即可查看。

不可刪除的 DOM 錨點：`#home-view`、`#homeNext`、`#homeTimeline`、`#lastModified`、`#moreQuickNav`、`#checklist`、`#map`、`#mapState`、`#backToSlotBtn`、`#mapDayTabs`、`#dayTabs`、`#dayCards`、`#pretripContent`、`#tipsList`、`#vjw-section`、`#vjwContent`、`#tax-refund-section`、`#taxRefundContent`、`#onsen-section`、`#onsenContent`、`#disclaimer-section` 與 `#disclaimer-text`。

### 入境與出境手續

- Day 1 提供前往「更多」Visit Japan Web 清單的按鈕；`taxRefund.departureDate` 對應的行程日提供退稅新制按鈕。以 `data-open-view` 和 `data-scroll-target` 切換 View 後捲動至指定區塊。
- Visit Japan Web 按準備階段分成可展開清單，勾選只供當次頁面自查，重新載入會重置；不使用本機儲存、不收集護照、帳密、個人 QR，也不向官方送出資料。此清單和不可勾選的「重要提醒」是不同功能。
- 入境／海關統一 QR、購物免稅 QR 與出境免稅品攜出確認是不同用途，不可混用；YKB Ensign Building 住宿已由使用者確認，入境住宿欄位依已確認的訂房憑證填寫，不再顯示舊地圖釘失效提醒。
- 退稅說明保留購物、整理行李、成田出境三階段，說明先付含稅價、托運前完成攜出確認與必要人工查驗、再由店家／業者返款。
- 新版資料的查核日期為 `2026-10-01`，購買適用日期為 `2026-11-01`，不是舊版誤寫的 12 月。保留官方 HTTPS 來源與行前再次確認提示；資料查核日期不等於此次整合日期。
- 1/4 保留約 `14:00` 抵達成田 T2 的緩衝建議，再依序查驗、托運、安檢與登機；不得宣稱一小時保證完成或機場立即現金退款。

## 4. 視覺與體驗方向

### 視覺語言

目前保留「雪夜冰藍，跨年金」的視覺語言，提供明亮與暗色兩種模式。每次開啟預設明亮模式，使用者可透過按鈕在當次頁面切換暗色模式；不讀取先前儲存的主題狀態。深藍、冰藍與金色仍作為封面及強調色，維持現有層次，不要改成高飽和漸層頁。

使用 CSS 變數控制顏色：`--bg`、`--bg-card`、`--text`、`--text-dim`、`--accent`、`--accent2`、`--border`、`--danger`、`--ok`。新增顏色時先評估是否可沿用既有變數。

### 排版與響應式

- 手機優先：主容器在窄螢幕採單欄，左右內距約 `1rem`。
- 桌面版：`768px` 以上容器擴至約 `920px`，地圖提高，適合的內容可以雙欄呈現。
- 日期 Tab 必須可水平滑動，按鈕不可因文字而壓縮變形。
- 卡片、景點、飯店或航班資訊需要可掃讀的層級，不在小面板中放過大的標題。
- 遠端圖片使用固定比例容器與 `object-fit: cover`；每張圖片都要保留載入失敗時的文字色塊。
- 首頁封面先宣告 `100vh` 高度備援，再宣告 `100svh`；保留 `-webkit-backdrop-filter` 與不透明背景供舊版 Safari 降級。三行摘要同時保留 `-webkit-line-clamp` 與標準 `line-clamp`，不支援時仍可閱讀完整內容。

### 互動原則

- 出發前預設 Day 1；旅途中預設當日；旅程結束後顯示最後一天回顧。
- 切換日期必須同時更新：行程與地圖 Tab 狀態、可見日卡、延遲圖片載入、景點的地圖編號、地圖標記、路線、地圖狀態文字。
- 行程中的「地圖 #N」是可操作按鈕；點擊後捲動至地圖並開啟同編號 marker 的 popup。
- Google 地圖導航是次要小型操作，避免在時間軸中搶過行程名稱與地圖編號。
- 使用 `kind: "restaurant"` 辨識餐廳，不依賴「晚餐：」等名稱；`showOnMap: false` 的餐廳不顯示地圖編號、Google 導航或 Leaflet marker，但仍保留 `lat/lng` 與訂位／官方連結。`showOnMap: true` 的餐廳（如江之島午餐）維持一般地圖導航。
- 底部導覽切換 View 後回到頁首、恢復顯示，並同步更新 active 狀態。
- 以 `target="_blank" rel="noopener"` 開啟外部地圖連結。

## 5. 資料模型：`trip-data` 是行程真實來源

`index.html` 內的下列區塊是完整行程資料來源，必須保持為合法 JSON：

```html
<script id="trip-data" type="application/json">
{ ... }
</script>
```

核心欄位如下。資料與互動以本文件及現行 `index.html` 為準；封存的通用契約只供歷史回溯，不作為新增欄位的必讀依據。

```text
trip
|- title, startDate, lastModified, colorScheme, coverPhoto, coverTitle, coverSubtitle, coverLocation
|- weatherLocations: { tokyo: {...}, nozawa: {...} }
|- preTrip: weather, packing, payment, apps, ticketTip
|- flights: booked[], candidates[]
|- hotelAreas[]: area, reason, options[]（含 checkIn、checkOut）
|- disclaimer, tips[], reminders[]
|- visitJapanWeb?: checkedOn, summary, privacyNote, sections[], sources[]
|- taxRefund?: checkedOn, effectiveDate, departureDate, summary, airportNote, sections[], sources[]
|- onsenGuide?: officialCheckedOn, summary, provenanceNote, safetyNote, planningNote, etiquette, baths[], sources[]
`- days[]
   |- date, weekday, theme, tips[], alternatives[], transportGuide?, ticketGuide?
   |- slots[]: period, kind?, showOnMap?, name, time, lat, lng, ...
   `- dining[]: meal, place, hours, dishes[]
```

每個 `slot` 至少應有：`period`、`name`、`time`、`needsBooking`、`leadDays`；列入地圖的節點須另有 `lat`、`lng`。座標一律 WGS-84；不可將 GCJ-02 或其他座標系直接寫入。僅帶有效 `onsenId` 且 `showOnMap: false` 的外湯可同時省略 `lat/lng`，表示尚無可靠定位；不得填入假座標或只省略其中一個。概略位置則附 `locationNote` 與 HTTPS `locationSource`，不可宣稱已核實入口或最短雪路。

`period` 使用 `morning`、`noon`、`evening` 或 `stay`。若 `needsBooking` 為 `true`，`leadDays` 應為大於零的合理天數，並同步考慮是否要在頂端 `reminders` 新增對應待辦。

可選景點欄位包括 `photo`、`rating`、`review`、`openingHours`、`closedDays`、`ticketPrice`、`transport`、`seasonal`。`transport` 可包含 `mode`、`fare`、`duration`；只有取得可靠官方來源時才加入 `actionLink`。

本頁專用欄位與缺值規則：

| 欄位 | 型別與語意 | 缺值時 |
| --- | --- | --- |
| `lastModified` | 必填 ISO 日期；頁尾顯示為八碼 `YYYYMMDD`，`<time datetime>` 保留完整 ISO 值，並與本文件第一行同步。 | 驗證失敗，須補上實際修改日。 |
| `coverPhoto`、`coverTitle`、`coverSubtitle`、`coverLocation` | 封面 HTTPS 圖片與文字；圖片載入失敗時隱藏。 | 標題退回 `title`；其餘留空並保留背景色。 |
| `weatherLocations` | 以 `tokyo`、`nozawa` 為鍵的物件；地點含 `label`、WGS-84 `lat/lng`、`seasonal`、可選 `tenkiUrl`。 | 無可用地點時不發請求；API 失敗顯示季節文字。 |
| 住宿 `checkIn`／`checkOut` | ISO 日期；含入住日、不含退房日。 | 退回當日住宿 slot 名稱。 |
| 每日 `transportGuide` | 可選交通攻略，含 `title`、`route`、`steps[]`、`sources[]` 等。 | 不顯示攻略。 |
| 每日 `ticketGuide` | 可選購票攻略，沿用交通攻略的步驟、提醒與官方來源格式。 | 不顯示攻略。 |
| `visitJapanWeb` | 可選入境準備資料；`checkedOn` 為 ISO 日期，`summary`、`privacyNote` 為文字；`sections[]` 含 `title`、字串陣列 `items[]`，`sources[]` 含 `label`、`url`。 | 隱藏 `#vjw-section` 及 Day 1 入口；去程航班缺值時省略航班參考。 |
| `taxRefund` | 可選出境退稅資料；`checkedOn`、`effectiveDate`、`departureDate` 為 ISO 日期，`summary`、`airportNote` 為文字；`sections[]`、`sources[]` 格式同上。 | 隱藏 `#tax-refund-section` 及出境日入口。 |
| `onsenGuide` | 可選 13 湯資料；`baths[]` 含唯一 `id`、`name`、`japaneseName`、`temperatureStatus`、`temperatureObserved`、`temperatureNote`、`visitPriority` 及可選目的、安全、替換、時段與休止提醒。`officialCheckedOn` 僅表示官方名稱、時段與公告核對日。 | 隱藏 `#onsen-section`；不產生巡湯附註與入口。 |
| 浴場 `temperatureSource` | 可選浴池水溫參考原文 HTTPS URL，顯示於表格各列的「紀錄與注意」。網路評論區間保留 `estimated` 狀態，不宣稱即時實測。 | 不顯示來源連結；缺水溫文字時仍以「待核實」安全降級。 |
| slot `onsenId` | 引用 `onsenGuide.baths[].id`，總覽與時間軸共用水溫資料；五日摘要從日期與 slot 推導，不另存重複計畫。 | 一般節點不顯示外湯資訊；無效引用由驗證器攔截。 |
| slot `locationNote`／`locationSource` | 定位限制及 HTTPS 來源；時間軸與全程／單日地圖彈窗均保留 `locationNote`。 | 未提供來源時不宣稱精準定位；來源存在時必須有說明。 |
| slot `kind`／`showOnMap` | `kind: "restaurant"` 表示餐廳；餐廳或帶 `onsenId` 的外湯可用 `showOnMap: false` 排除地圖標記、編號及導航。 | 一般節點與外湯預設列入地圖；餐廳只有明確 `showOnMap: true` 才列入。 |
| 外部 URL | `coverPhoto`、`photo`、`tenkiUrl`、`actionLink.url`、`locationSource`、交通／購票攻略、入出境及外湯來源只允許絕對 `https:` URL。 | 非 HTTPS 或格式錯誤的連結不渲染。 |

## 6. 資料同步規則

`trip-data` 是唯一行程資料來源。Hero 日期、日期 Tab、首頁、行前須知、航班、住宿、Visit Japan Web、退稅新制、外湯參考與五日巡湯、免責聲明、提醒與全程 Tips 都在載入時由 JavaScript 產生；修改行程時只更新 JSON，不得重新加入手寫資料副本。

頁面仍保留少量沒有業務資料的 HTML 骨架，例如區塊標題、渲染容器與底部導覽。若新增資料欄位，應在 `renderTripSummary()`、`renderHome()` 或每日時間軸 renderer 中集中處理。

### 2027/1/2–1/4 住宿與動線

使用者已確認東京六本木雷姆飯店（remm Roppongi）塔景房兩晚，1/2 入住、1/4 退房。正式資料需同步更新 `hotelAreas`、兩晚住宿 slot、入住／退房與交通 slot；原先「後兩晚飯店待訂」及已訂飯店的憑證核對提醒均已移除，不再列入行前提醒。首頁、時間軸與地圖均由更新後的 JSON 渲染。不得推定房數、房價或早餐，也不要把三人同行誤寫成單間房可容納三人。

| 日期 | 順序與優先事項 |
| --- | --- |
| 1/2 | 保留飯山初詣、鶴屋與午餐 → 取行李 → 北陸新幹線返東京 → 地鐵至六本木 → 飯店入住放行李 → 新宿初賣（看營業與體力，可略過）→ 六本木晚餐與塔景房休息。 |
| 1/3 | 早餐後 08:30 六本木出發 → 明治神宮（09:15–10:30，可跳過）→ 經原宿／新宿到吉祥寺（目標 11:45 前）→ 午餐 → 吉卜力美術館（目標 13:00，依購票時段）→ 井之頭公園（視體力與天色）→ 返回六本木晚餐與住宿。 |
| 1/4 | 六本木提早退房並攜全部行李 → 東京站寄物與補貨 → 午餐 → 取行李與候車 → 約 14:00 抵成田 T2 → 免稅品攜出確認 → 托運、安檢、登機；不折返飯店。 |

飯店地址與交通於 2026-10-10 核對[飯店官網](https://www.hankyu-hotel.com/hotel/remm/roppongi/access)：東京都港區六本木 7-14-4，日比谷線 4a 出口／大江戶線 7 號出口步行約 1 分鐘。頁面地圖採官網地圖中心座標 `35.66391, 139.731292`；攜行李時另依現場電梯出口指引。新幹線、地鐵及入住時間均為行程預留，實際依車次與訂房憑證，不使用高田馬場作為住宿出發點。

### 12/28–1/2 長野／野澤孕婦友善與雙軌行程

考量同行者屆時懷孕約 5 個月，野澤溫泉停留期間（12/28–1/2）採「滑雪組＋孕婦慢遊組雙軌」與「全員非滑雪一日遊」交錯設計：

- **12/28（Day 3）**：上午新幹線與巴士入村、寄放行李並依實際滑雪天數（12/28 下午、12/29、12/31）租借雪具；下午雙軌並列「日影・上之平半日滑」與「孕婦村內咖啡廳休息／長坂 Gondola 景觀餐廳會合」。
- **12/29（Day 4）**：白天雙軌並列「Yamabiko 粉雪・上之平巡航」與「孕婦麻釜熱湯湧泉外觀賞・溫泉街咖啡廳慢遊」，中午可搭密閉式長坂 Gondola 至山頂 Rest House Yamabiko 共進午餐。
- **12/30（Day 5）**：全體改為「地獄谷野猿公苑一日遊」。提供 `transportGuide` 說明野澤 ⇄ 飯山 ⇄ 上林溫泉大眾運輸與局部計程車彈性接駁（減少孕婦吹風候車）、必備鞋底防滑冰爪（Crampons）、入口「猿座咖啡（Enza Cafe）」暖氣午餐與無障礙洗手間，以及風雪過大時改逛澀溫泉古街的備案。
- **12/31（Day 6）**：白天最後一日滑雪與孕婦村內慢遊雙軌；下午辦理雪具歸還（參加夜滑者確認夜滑歸還規定）。晚間提早於 17:30 用餐，19:45–21:00 以「大晦日長坂夜滑・跨年煙火」為主案，原訂坡道較陡的滝湯改列為不參加夜滑時的彈性備案；23:30 接續除夜之鐘。
- **1/1（Day 7）**：全體改為「長野善光寺周遭一日遊」。提供 `transportGuide` 說明野澤 ⇄ 飯山 ⇄ JR 長野站（北陸新幹線僅約 11 分鐘）及長野站 ⇄ 善光寺計程車／巴士接駁；包含善光寺初詣與求安產御守（提醒孕婦避開推擠與全黑狹窄的戒壇巡禮）、仲見世通美食散策、城山公園／長野縣立美術館周邊平地休息，以及 JR 長野站 MIDORI 商場室內採買與午茶。
- **1/2（Day 8）**：維持飯山初詣、鶴屋採買與北陸新幹線返東京入住六本木。

### 野澤五日巡湯

依使用者最新安排，12/28–1/1（全程 Day 3–7）每天晚餐後安排一間主浴場（12/31 晚間以大晦日長坂夜滑與跨年煙火為主，滝湯作為不夜滑時的備案），五天不重複。餐後至少保留 15 分鐘起始緩衝，仍須視身體狀況休息；飲酒後不泡湯，孕婦亦不建議浸泡高溫公共外湯（改以民宿溫水淋浴或外觀巡禮為主）。時段包含步行、換衣與休息，不是連續浸泡時間。晚餐延誤、水溫、路況或開放狀況不合就取消，不為打卡趕路。

| 日期 | 主浴場 | 晚餐後預留時段 |
| --- | --- | --- |
| 12/28 | 大湯 | 20:45–21:45；22:00 後返回住宿休息。 |
| 12/29 | 熊手洗湯 | 20:15–21:15；21:30 後返回住宿休息。 |
| 12/30 | 真湯（可跳過） | 20:45–21:30；一般 22:00 關閉，預留換衣離場時間。 |
| 12/31 | 大晦日夜滑＆煙火（備案：滝湯） | 19:45–21:00；以長坂夜滑與跨年煙火為主，滝湯列為不夜滑備案，21:00 前回住宿休息，23:30 參加除夜之鐘。 |
| 1/1 | 中尾湯 | 20:45–22:00；22:15 後返回住宿並整理隔日行李。 |

水溫綜合使用者提供的旅客紀錄摘要及網路評論的約略值／區間。`temperatureStatus` 的 `reported`、`estimated`、`unverified` 分別表示摘要所述紀錄、估測／評論、無數據，**不表示即時實測或官方固定值**。`temperatureNote` 保留加水、浴池與可信度限制；既有摘要部分原始量測日期仍不明。依使用者最新要求，秋葉、松葉、新田、橫落、十王堂已用查閱到的浴池評論區間取代「待核實」，不以泉源溫度或其他浴場補值。只有單次約值時不杜撰區間上下限；將來真正缺資料時仍使用「待核實」降級。共用及個別 `safetyNote` 均須可見，約 43–44°C 也不保證適合所有人。

| 浴場 | 網路浴池水溫參考 | 來源與限制 |
| --- | --- | --- |
| 秋葉湯 | 約 41–49°C | [旅客評論](https://onsen.nifty.com/iiyama-nagano-onsen/onsen007667/kuchikomi/)含冷泉調溫後約 41–42°C、約 42°C 與約 49°C 加水後仍約 47°C 的紀錄；保留休止公告，不代表已恢復開放。 |
| 松葉湯 | 約 43–47°C | [旅客評論](https://onsen.nifty.com/iiyama-nagano-onsen/onsen007674/kuchikomi/)含約 43°C、約 47°C 的浴池紀錄；不以湯色判斷水溫。 |
| 新田湯 | 約 43–44°C | [旅客評論](https://onsen.nifty.com/iiyama-nagano-onsen/onsen007670/kuchikomi/)含加水後約 43°C 與接近 44°C；不保證未加水時也如此。 |
| 橫落湯 | 約 43–44°C（體感） | [旅客評論](https://onsen.nifty.com/iiyama-nagano-onsen/onsen007675/kuchikomi/)另有約 50°C 供給水，不能混作整池實測上限；保留實際可能更燙的警示。 |
| 十王堂湯 | 約 43–48°C | [旅客評論](https://onsen.nifty.com/iiyama-nagano-onsen/onsen007666/kuchikomi/)含約 44–46°C、約 48°C 與加水後約 43°C；不同時期與浴池不可視為同一測試。 |

「13 湯水溫與替換選項」使用語意化三欄表格：浴場、約略水溫、選擇／替換。每列保留浴場優先度與直接可讀的水溫，日文名稱、紀錄日期、個別風險及官方時段放在「紀錄與注意」展開區；有休止公告者收合時也顯示「開放狀態待確認」。表格由同一份 `onsenGuide.baths` 渲染，不新增資料副本。採 `table-layout: fixed`、自動換行及較小手機內距，320px 手機不需橫向捲動；展開按鈕至少 44px 高。總覽上方的共用高溫警示不收合，逐列詳細內容保持可存取，每日時間軸的完整安全提醒不變。

[官方 13 湯指南](https://nozawakanko.jp/hotspring/)與[外湯地圖](https://nozawakanko.jp/wp/wp-content/themes/nozawa_kanko/assets/img/hotspring/pdf/sotoyu_map.pdf)只核對浴場名稱、時段、位置及禮儀，不是旅客浴池水溫證據。麻釜浴場與麻釜泉源不能混淆，非村民不得進入泉源區。[秋葉湯公告](https://nozawakanko.jp/13524/)預計休止至 2026/10/30，不能據此宣稱冬季已恢復開放。

五間主浴場已加入概略定位：大湯、熊手洗湯及真湯採 OpenStreetMap 具名點位；滝湯採 Google 地圖浴場點位，中尾湯採具名地圖中心與村南側相對位置。來源 URL 保留於各 slot 的 `locationSource`，`locationNote` 明示不保證浴場入口或最短冬季步行路線。巡湯總覽的 `data-select-day` 按鈕同步切換行程及地圖日期；每日節點提供前往「更多」完整參考資料的入口。

## 7. 不可破壞的程式契約

### 內聯引擎

發佈用單檔 HTML 必須保留內聯的地圖與提醒函式；Leaflet CSS／JS 及圖磚則由網路提供：

- `escapeHTML`：輸出動態文字前的跳脫。
- `buildMapAppLinks`：為每個景點提供單一 Google 地圖導航入口。
- `initTravelMap`：初始化 Leaflet，並提供 `map.setTravelPoints()`。
- `computeReminders` 與 `renderChecklistHTML`：計算並產生不可勾選的重要提醒；函式名稱為相容既有驗證器而保留。
- `reminderBadgeHTML`：顯示需預訂景點的提醒。

封存技能的 `map.js` 與 `reminders.js` 各有獨立的 `escapeHTML`；現行 `index.html` 內聯版本只宣告一份共用函式。維護本頁不須同步修改封存檔案，也不要因此誤刪頁面中的 `escapeHTML`。

Leaflet CDN 失敗或瀏覽器處於離線狀態時，地圖容器顯示文字提示並回傳相容的無作用物件，首頁、三 View、每日行程與提醒仍可正常操作。Leaflet 已載入但圖磚失敗時顯示圖磚提示，不清除標記與路線；天氣 API 失敗時保留季節參考文字。

### 地圖與日切換

1. 排除明確 `showOnMap: false` 的餐廳／外湯，以及未明確 `showOnMap: true` 的餐廳；其餘 `days[].slots[]` 依原始日期及景點順序彙整為全行程 `points`。
2. 全行程模式使用跨日遞增的「地圖 #N」。
3. 單日模式只傳入該日點位，並從「地圖 #1」重新編號。
4. 日卡片標題旁的編號必須和目前地圖標記一致。
5. 有地圖標記的 slot 卡與 Leaflet 彈窗保留 Google 地圖導航及可選概略定位提醒；未加入地圖的餐廳仍可提供合法的訂位／官方連結，無可靠定位的外湯不提供座標導航。

### 安全與內容

- 所有動態文字都經過 `escapeHTML` 或使用 `textContent`；不可直接拼接未跳脫的文字到 `innerHTML`。
- `escapeHTML` 無法驗證 URL；資料提供的外部 URL 必須先用 `safeExternalURL` 驗證為絕對 HTTPS 再渲染。
- 不自行捏造訂票、叫車或即時資料連結；`actionLink` 僅使用可信工具或官方來源回傳的 URL。
- 票價、營業時間、天氣、評分等屬參考資訊，需維持顯著免責聲明。
- 使用繁體中文；日文或英文僅用於地名、店名和專有名詞。

## 8. VS Code 開發流程

1. 變更前對照本文件與 `index.html`，確認要改的 JSON 欄位、渲染函式及受影響的 View。
2. 只在 `trip-data` 維護行程內容；新增欄位時再調整 `renderTripSummary()`、`renderHome()`、每日時間軸或地圖的渲染邏輯，不建立手寫資料副本。
3. 以局部修改保留單檔架構；核對日期、WGS-84 座標、跨午夜時間、交通緩衝與 HTTPS 連結。
4. 執行結構驗證及瀏覽器回歸測試；若失敗先修正再提交。票價與營業資訊仍應人工核實。

## 9. 驗證清單

首次測試先在 `test-only/` 安裝依賴（Windows 測試使用本機 Chrome）。從工作區根目錄執行：

```powershell
npm --prefix ./test-only ci
npm --prefix ./test-only test
```

安裝只需做一次；之後每次改動執行 `npm --prefix ./test-only test`，它會先執行 `test-only/validate.cjs` 檢查根目錄頁面，再執行瀏覽器回歸測試。只想跑結構檢查則使用 `npm --prefix ./test-only run validate`。

結構驗證器只檢查 JSON、座標及少數標記，**通過不代表互動行為正確**。瀏覽器測試使用本機 Leaflet、模擬天氣資料並阻斷外部網路，涵蓋三 View、離線與 CDN／圖磚失敗、雙 Tab 與地圖 popup、日本時間與跨午夜、不安全連結及餐廳規則。

新版整合的驗證範圍還包含：所有可執行內聯 JavaScript 的語法、購票攻略／入出境手續的日期與 HTTPS 來源、Day 1／出境日的跨 View 入口、VJW 勾選重新載入重置、可選資料缺值與頁底導覽恢復。文件的程式碼圍欄、表格與欄位名稱也須保持一致；Chrome 測試不能取代實機 Safari 相容性驗證。

外湯驗證包含 13 湯唯一 ID、五筆網路參考區間與 HTTPS 原文來源、五日不重複且位於晚餐後、住宿與跨年時段順序、概略點位的全程／單日地圖警示、雙向 View 入口、安全跳脫及缺值降級；真正缺值仍不補數字，來源連結拒絕非 HTTPS。表格另檢查 13 列、列／欄標題語意、逐列展開、44px 觸控高度與 320px／390px／桌面不溢出。現有測試集中於同一檔案，不新增另一份 HTML 或行程資料副本。

驗證通過後，至少人工確認：

- `trip-data` 是合法 JSON，且標題、日期、航班、住宿沒有和靜態區塊矛盾。
- 新增或搬移的 slot 有 WGS-84 `lat`、`lng`，且位於合理地理位置。
- Day Tab 數量與 `trip.days` 數量相符；目前由 JavaScript 動態產生。
- 全行程與單日模式的地圖標記、路線和「地圖 #N」相同。
- 圖片載入失敗時僅隱藏圖片，仍可看到景點名稱的備援底圖。
- 手機寬度下底部導覽、Tab、卡片文字不溢出；桌面寬度下內容不會窄成單一長欄。

## 10. 修改範圍判斷

適合直接處理的項目：更換景點、調整時間、增減餐飲、補充景點資訊、更新已訂航班或住宿、調整單一段落樣式、修正地圖座標、增加一個現有模式可容納的欄位。

應拆成獨立任務的項目：更換地圖供應商、改為多檔案發佈架構、導入前端框架、改造整體視覺主題、加入後端或即時訂票功能。

任何不確定的票價、營業時間、節慶活動或預訂狀態，都應在頁面保留「參考，請自行核實」的語意，不能當作即時事實宣告。

## 11. 新版整合記錄

2026-10-10 完成另一台電腦新版與本機正式頁的比對整合：新版保留三 View、每日行程與地圖契約，新增 VJW 與退稅資料／區塊／快捷入口、修正退稅適用日期與 1/4 托運前順序、整理全程 Tips、加入 Safari CSS 備援及頁底導覽恢復。免責聲明移至「更多」最後並改由 JSON 渲染，不保留靜態文字副本。依使用者要求，整合後刪除新版比較檔及兩份封存 HTML，只保留 `index.html`；設計文件、測試、工作區設定及非 HTML 封存資料仍保留。
