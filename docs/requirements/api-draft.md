# API 草稿

## 1. 文件目的

本文件用於整理 **多人協作自由行規劃網站** 在 MVP 階段所需的 API 分類、端點用途、建議 HTTP Method、權限範圍，以及 request / response 的基本設計方向。

本文件應搭配以下文件一起閱讀：

- `docs/requirements/mvp-requirements.md`
- `docs/requirements/pages-and-flows.md`
- `docs/requirements/data-model-draft.md`

---

## 2. API 設計原則

### 2.1 MVP 優先

API 只涵蓋 MVP 必做功能：

- 建立旅程
- 邀請成員加入旅程
- 建立與編輯個人行程
- 查看其他成員行程
- 查看全員總表
- 設定共同行程
- 複製其他成員某一天行程到自己的個人行程
- 維護住宿基本資訊

### 2.2 權限由旅程角色控制

API 權限需依據使用者在該旅程中的角色判定：

- 團主
- 成員
- 只讀成員

### 2.3 REST 為主

MVP 階段以 REST API 為主，不先引入複雜的即時同步機制。

### 2.4 回應格式一致

建議所有 API 回應維持一致格式，例如：

```json
{
  "success": true,
  "message": "操作成功",
  "data": {}
}
```

錯誤格式建議：

```json
{
  "success": false,
  "message": "錯誤說明",
  "errors": []
}
```

---

## 3. 權限摘要

| 角色 | 可查看旅程 | 可查看他人行程 | 可編輯自己行程 | 可管理成員 | 可設定共同行程 | 可管理住宿 |
|---|---|---|---|---|---|---|
| 團主 | 是 | 是 | 是 | 是 | 是 | 是 |
| 成員 | 是 | 是 | 是 | 否 | 依團主授權 | 否 |
| 只讀成員 | 是 | 是 | 否 | 否 | 否 | 否 |

### 備註

- 團主是否可直接編輯他人的個人行程，仍列為待確認事項
- 成員在獲得團主授權後，可建立、編輯、刪除共同行程

---

## 4. API 分類

MVP API 建議分為以下幾類：

1. 認證與使用者
2. 旅程管理
3. 成員管理
4. 個人行程管理
5. 共同行程管理
6. 全員總表
7. 行程複製
8. 住宿管理

---

## 5. 認證與使用者 API

### 5.1 取得目前使用者資訊

**Endpoint**  
`GET /api/me`

**用途**  
取得目前登入者基本資訊。

**權限**  
- 已登入使用者

**Response 重點**
- 使用者 Id
- 顯示名稱
- Email

---

## 6. 旅程管理 API

### 6.1 取得我的旅程列表

**Endpoint**  
`GET /api/trips`

**用途**  
取得目前使用者參與的所有旅程列表。

**權限**  
- 已登入使用者

**Query 建議**
- `keyword`：關鍵字搜尋
- `status`：旅程狀態（可選）

**Response 重點**
每筆旅程可包含：
- TripId
- Name
- StartDate
- EndDate
- Role
- MemberCount

### 6.2 建立旅程

**Endpoint**  
`POST /api/trips`

**用途**  
建立新旅程，建立者自動成為團主。

**權限**  
- 已登入使用者

**Request 建議**

```json
{
  "name": "東京自由行",
  "description": "2026 東京 5 天 4 夜",
  "startDate": "2026-07-10",
  "endDate": "2026-07-14"
}
```

**Response 重點**
- 新旅程 Id
- 建立成功訊息
- 團主角色資訊

### 6.3 取得單一旅程詳細資料

**Endpoint**  
`GET /api/trips/{tripId}`

**用途**  
取得單一旅程的基本資訊與摘要。

**權限**
- 團主
- 成員
- 只讀成員

**Response 重點**
- 旅程基本資料
- 我的角色
- 成員數
- 住宿數
- 日期範圍

### 6.4 更新旅程基本資料

**Endpoint**  
`PUT /api/trips/{tripId}`

**用途**  
更新旅程基本資料。

**權限**
- 團主

**Request 建議**

```json
{
  "name": "東京自由行",
  "description": "更新後描述",
  "startDate": "2026-07-10",
  "endDate": "2026-07-14"
}
```

### 6.5 刪除旅程

**Endpoint**  
`DELETE /api/trips/{tripId}`

**用途**  
刪除整個旅程。

**權限**
- 團主

**備註**
MVP 可先實作軟刪除或狀態停用，不一定真的物理刪除。

---

## 7. 成員管理 API

### 7.1 取得旅程成員列表

**Endpoint**  
`GET /api/trips/{tripId}/members`

**用途**  
取得旅程中的所有成員與角色。

**權限**
- 團主
- 成員
- 只讀成員

**Response 重點**
每位成員包含：
- UserId
- DisplayName
- Email
- Role
- JoinedAt

### 7.2 邀請成員加入旅程

**Endpoint**  
`POST /api/trips/{tripId}/members`

**用途**  
新增旅程成員。

**權限**
- 團主

**Request 建議**

```json
{
  "userId": "target-user-id",
  "role": "member"
}
```

**備註**
- MVP 不先綁定 Email 邀請流程
- 可先用系統內選人或直接指定使用者方式

### 7.3 更新成員角色

**Endpoint**  
`PUT /api/trips/{tripId}/members/{memberId}`

**用途**  
調整旅程成員角色。

**權限**
- 團主

**Request 建議**

```json
{
  "role": "readonly"
}
```

### 7.4 移除成員

**Endpoint**  
`DELETE /api/trips/{tripId}/members/{memberId}`

**用途**  
將某位成員從旅程中移除。

**權限**
- 團主

---

### 7.5 更新成員共同行程權限

**Endpoint**  
`PUT /api/trips/{tripId}/members/{memberId}/shared-activity-permission`

**用途**  
由團主設定某位成員是否可管理共同行程。

**權限**
- 團主

**Request 建議**

```json
{
  "canManageSharedActivities": true
}

備註

此權限僅控制共同行程管理能力
不影響該成員是否可編輯他人的個人行程

---

## 8. 個人行程管理 API

### 8.1 取得自己的某日個人行程

**Endpoint**  
`GET /api/trips/{tripId}/my-itinerary?date=2026-07-10`

**用途**  
取得目前登入者在指定日期的個人行程。

**權限**
- 團主
- 成員

**Response 重點**
- 日期
- 個人行程主資料
- 該日行程項目列表

### 8.2 取得指定成員某日個人行程

**Endpoint**  
`GET /api/trips/{tripId}/members/{userId}/itinerary?date=2026-07-10`

**用途**  
查看指定成員在指定日期的個人行程。

**權限**
- 團主
- 成員
- 只讀成員

**備註**
- 僅查看，不允許透過此 API 修改他人個人行程

### 8.3 建立自己的某日個人行程主資料

**Endpoint**  
`POST /api/trips/{tripId}/my-itineraries`

**用途**  
建立某一天的個人行程容器。

**權限**
- 團主
- 成員

**Request 建議**

```json
{
  "date": "2026-07-10"
}
```

**備註**
若系統採自動建立，也可省略此 API。

### 8.4 新增個人行程項目

**Endpoint**  
`POST /api/trips/{tripId}/my-itineraries/{itineraryId}/items`

**用途**  
新增個人行程中的單一項目。

**權限**
- 團主
- 成員（限新增到自己的個人行程）

**Request 建議**

```json
{
  "title": "淺草寺",
  "category": "sightseeing",
  "startTime": "09:00",
  "endTime": "11:00",
  "locationName": "淺草寺",
  "address": "東京都台東區...",
  "notes": "早上先去參拜"
}
```

### 8.5 更新個人行程項目

**Endpoint**  
`PUT /api/trips/{tripId}/my-itineraries/{itineraryId}/items/{itemId}`

**用途**  
更新個人行程中的單一項目。

**權限**
- 團主
- 成員（限自己的個人行程）

### 8.6 刪除個人行程項目

**Endpoint**  
`DELETE /api/trips/{tripId}/my-itineraries/{itineraryId}/items/{itemId}`

**用途**  
刪除個人行程中的單一項目。

**權限**
- 團主
- 成員（限自己的個人行程）

### 8.7 調整個人行程項目排序

**Endpoint**  
`PUT /api/trips/{tripId}/my-itineraries/{itineraryId}/items/reorder`

**用途**  
更新某日個人行程項目的順序。

**權限**
- 團主
- 成員（限自己的個人行程）

**Request 建議**

```json
{
  "items": [
    { "itemId": "item-1", "sortOrder": 1 },
    { "itemId": "item-2", "sortOrder": 2 }
  ]
}
```

---

## 9. 共同行程管理 API

### 9.1 取得某日共同行程列表

**Endpoint**  
`GET /api/trips/{tripId}/shared-activities?date=2026-07-10`

**用途**  
取得指定日期的共同行程列表。

**權限**
- 團主
- 成員
- 只讀成員

### 9.2 新增共同行程

**Endpoint**  
`POST /api/trips/{tripId}/shared-activities`

**用途**  
建立某一天或某個時段的共同行程。

**權限**
- 團主
- 已獲授權可管理共同行程的成員

**Request 建議**

```json
{
  "date": "2026-07-10",
  "title": "全員一起去晴空塔",
  "description": "下午一起集合",
  "startTime": "14:00",
  "endTime": "17:00",
  "locationName": "東京晴空塔",
  "address": "東京都墨田區..."
}
```

### 9.3 更新共同行程

**Endpoint**  
`PUT /api/trips/{tripId}/shared-activities/{sharedActivityId}`

**用途**  
更新既有共同行程。

**權限**
- 團主
- 已獲授權可管理共同行程的成員

### 9.4 刪除共同行程

**Endpoint**  
`DELETE /api/trips/{tripId}/shared-activities/{sharedActivityId}`

**用途**  
刪除既有共同行程。

**權限**
- 團主
- 已獲授權可管理共同行程的成員

### 9.5 設定共同行程參與成員

**Endpoint**  
`PUT /api/trips/{tripId}/shared-activities/{sharedActivityId}/participants`

**用途**  
指定哪些成員參與該共同行程。

**權限**
- 團主

**Request 建議**

```json
{
  "participantUserIds": [
    "user-1",
    "user-2",
    "user-3"
  ]
}
```

**備註**
若 MVP 先採全部成員預設參與，可暫不實作此 API。

---

## 10. 全員總表 API

### 10.1 取得某日全員總表

**Endpoint**  
`GET /api/trips/{tripId}/group-schedule?date=2026-07-10`

**用途**  
取得指定日期的全員總表資料。

**權限**
- 團主
- 成員
- 只讀成員

**Response 重點**
建議包含：
- 日期
- 共同行程列表
- 每位成員的個人行程摘要
- 行程類型標記（shared / personal）

**Response 範例概念**

```json
{
  "date": "2026-07-10",
  "sharedActivities": [],
  "members": [
    {
      "userId": "user-1",
      "displayName": "Nick",
      "personalItems": []
    }
  ]
}
```

---

## 11. 行程複製 API

### 11.1 複製其他成員某一天行程到自己

**Endpoint**  
`POST /api/trips/{tripId}/itinerary-copy`

**用途**  
將其他成員某一天的個人行程複製到目前登入者自己的個人行程。

**權限**
- 團主
- 成員

**Request 建議**

```json
{
  "sourceUserId": "source-user-id",
  "sourceDate": "2026-07-10",
  "targetDate": "2026-07-10"
}
```

**MVP 規則**
- 第一版先支援整天複製
- 只允許複製到自己的個人行程
- 不支援部分項目複製

**Response 重點**
- 複製成功訊息
- 新增或更新後的目標個人行程摘要

---

## 12. 住宿管理 API

### 12.1 取得住宿列表

**Endpoint**  
`GET /api/trips/{tripId}/lodgings`

**用途**  
取得該旅程的住宿資訊列表。

**權限**
- 團主
- 成員
- 只讀成員

### 12.2 新增住宿資訊

**Endpoint**  
`POST /api/trips/{tripId}/lodgings`

**用途**  
新增一筆住宿資料。

**權限**
- 團主

**Request 建議**

```json
{
  "name": "東京站飯店",
  "address": "東京都千代田區...",
  "checkInDate": "2026-07-10",
  "checkOutDate": "2026-07-12",
  "notes": "近車站",
  "roomInfo": "雙人房"
}
```

### 12.3 更新住宿資訊

**Endpoint**  
`PUT /api/trips/{tripId}/lodgings/{lodgingId}`

**用途**  
更新既有住宿資料。

**權限**
- 團主

### 12.4 刪除住宿資訊

**Endpoint**  
`DELETE /api/trips/{tripId}/lodgings/{lodgingId}`

**用途**  
刪除一筆住宿資料。

**權限**
- 團主

---

## 13. 常見狀態碼建議

| 狀態碼 | 用途 |
|---|---|
| 200 OK | 查詢或更新成功 |
| 201 Created | 建立成功 |
| 400 Bad Request | 請求格式錯誤或欄位驗證失敗 |
| 401 Unauthorized | 未登入 |
| 403 Forbidden | 已登入但沒有權限 |
| 404 Not Found | 找不到資源 |
| 409 Conflict | 資料衝突，例如重複加入成員 |
| 500 Internal Server Error | 系統錯誤 |

---

## 14. 欄位驗證建議

### 14.1 Trip
- Name：必填，長度合理限制
- StartDate / EndDate：必填
- EndDate 不可早於 StartDate

### 14.2 TripMember
- Role：必須為 owner / member / readonly
- 同一旅程不可重複加入同一使用者

### 14.3 PersonalItineraryItem
- Title：必填
- Category：必填
- StartTime / EndTime：若都有值，EndTime 不可早於 StartTime

### 14.4 SharedActivity
- Date：必填
- Title：必填

### 14.5 Lodging
- Name：必填
- Address：必填
- CheckOutDate 不可早於 CheckInDate

---

## 15. 待確認事項

以下 API 細節仍需後續確認：

- 團主是否可直接修改他人的個人行程
- 住宿資訊是否只允許團主管理
- 行程複製後是否要記錄來源資訊給前端顯示
- 是否保留獨立的 TripDay API
- 是否需要額外提供首頁 Dashboard 摘要 API

---

## 16. 下一步建議

完成本文件後，建議下一步進入以下其中一項：

1. 實際 SQL Server 資料表設計
2. ASP.NET Core Web API 專案骨架
3. 前端頁面骨架與路由規劃
