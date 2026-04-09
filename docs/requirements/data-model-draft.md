# 資料模型草稿

## 1. 文件目的

本文件用於整理 **多人協作自由行規劃網站** 在 MVP 階段的核心資料實體、建議欄位與實體之間的關係。

本文件為資料模型草稿，目標是先建立清楚的資料概念，後續再轉成實際的 SQL Server 資料表設計。

本文件應搭配以下文件一起閱讀：

- `docs/requirements/mvp-requirements.md`
- `docs/requirements/pages-and-flows.md`

---

## 2. 資料模型設計原則

### 2.1 MVP 優先原則
資料模型先支援 MVP 必做功能，不預設加入第二版功能，例如：

- 留言
- 預算追蹤
- 通知
- Google Maps
- 天氣 API

### 2.2 個人行程與共同行程分開思考
雖然未來實作時可以共用部分資料表，但在業務概念上需清楚區分：

- 個人行程
- 共同行程

### 2.3 手機與電腦共用同一份資料
手機版與電腦版只是呈現方式不同，底層資料模型應盡量一致。

### 2.4 權限由旅程成員關係決定
誰能看、誰能改，不應直接寫死在行程資料本身，而應主要由：

- 使用者
- 旅程
- 成員角色

之間的關係決定。

---

## 3. 核心資料實體

MVP 階段建議的核心資料實體如下：

1. User
2. Trip
3. TripMember
4. TripDay
5. PersonalItinerary
6. PersonalItineraryItem
7. SharedActivity
8. SharedActivityParticipant
9. Lodging
10. ItineraryCopyLog

---

## 4. 實體說明與建議欄位

## 4.1 User

### 用途
代表系統中的註冊使用者。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 使用者主鍵 |
| DisplayName | nvarchar(100) | 是 | 顯示名稱 |
| Email | nvarchar(255) | 是 | 使用者登入或識別用 |
| PasswordHash | nvarchar(500) | 否 | 若採帳密登入則需要 |
| CreatedAt | datetime2 | 是 | 建立時間 |
| UpdatedAt | datetime2 | 是 | 更新時間 |
| IsActive | bit | 是 | 是否啟用 |

### 備註
- 若登入方式之後改為第三方登入，`PasswordHash` 可再調整
- MVP 階段先保留最基本使用者資料即可

---

## 4.2 Trip

### 用途
代表一趟旅程，是整個規劃的主容器。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 旅程主鍵 |
| Name | nvarchar(200) | 是 | 旅程名稱 |
| Description | nvarchar(max) | 否 | 簡短描述 |
| StartDate | date | 是 | 旅程開始日期 |
| EndDate | date | 是 | 旅程結束日期 |
| OwnerUserId | uniqueidentifier | 是 | 團主使用者 Id |
| CreatedAt | datetime2 | 是 | 建立時間 |
| UpdatedAt | datetime2 | 是 | 更新時間 |
| Status | nvarchar(50) | 否 | 旅程狀態，例如 Draft / Active |

### 備註
- `OwnerUserId` 對應旅程建立者
- `Status` 在 MVP 可先簡化或不實作

---

## 4.3 TripMember

### 用途
表示某位使用者參與某趟旅程，以及其在該旅程中的角色。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| TripId | uniqueidentifier | 是 | 所屬旅程 |
| UserId | uniqueidentifier | 是 | 所屬使用者 |
| Role | nvarchar(50) | 是 | owner / member / readonly |
| JoinedAt | datetime2 | 是 | 加入時間 |
| InvitedByUserId | uniqueidentifier | 否 | 邀請人 |
| IsActive | bit | 是 | 是否仍為此旅程成員 |
| CanManageSharedActivities | bit | 是 | 是否可設定共同行程，預設為 false |

### 備註
- `TripId + UserId` 應避免重複
- 角色權限建議由這張表決定
- 除了 Role 之外，也可透過額外權限欄位控制成員是否能管理共同行程

---

## 4.4 TripDay

### 用途
表示旅程中的某一天，方便總表、每日行程與日期切換。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| TripId | uniqueidentifier | 是 | 所屬旅程 |
| Date | date | 是 | 該旅程中的某一天 |
| DayNumber | int | 否 | 第幾天，例如 Day 1 |
| CreatedAt | datetime2 | 是 | 建立時間 |

### 備註
- MVP 可選擇是否實作這張表
- 若不建立，也可直接用 `Trip + Date` 表示某一天
- 若你想讓每日資料管理更清楚，建議保留

---

## 4.5 PersonalItinerary

### 用途
代表某位成員在某趟旅程中的某一天個人行程。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| TripId | uniqueidentifier | 是 | 所屬旅程 |
| UserId | uniqueidentifier | 是 | 行程擁有者 |
| TripDayId | uniqueidentifier | 否 | 對應旅程日期 |
| Date | date | 是 | 行程日期 |
| CreatedAt | datetime2 | 是 | 建立時間 |
| UpdatedAt | datetime2 | 是 | 更新時間 |

### 備註
- 一位使用者在同一旅程同一天通常只會有一筆 PersonalItinerary
- 真正的行程項目放在 `PersonalItineraryItem`

---

## 4.6 PersonalItineraryItem

### 用途
代表個人行程中的單一項目，例如景點、餐廳、交通、活動。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| PersonalItineraryId | uniqueidentifier | 是 | 所屬個人行程 |
| Title | nvarchar(200) | 是 | 項目標題 |
| Category | nvarchar(50) | 是 | sightseeing / dining / transport / activity / lodging / other |
| StartTime | time | 否 | 開始時間 |
| EndTime | time | 否 | 結束時間 |
| LocationName | nvarchar(255) | 否 | 地點名稱 |
| Address | nvarchar(500) | 否 | 地址 |
| Notes | nvarchar(max) | 否 | 備註 |
| SortOrder | int | 是 | 顯示排序 |
| CreatedAt | datetime2 | 是 | 建立時間 |
| UpdatedAt | datetime2 | 是 | 更新時間 |

### 備註
- MVP 先不必拆得太細
- `Category` 可以先用字串或 enum 管理
- 若沒有時間，也能只靠 `SortOrder` 排序

---

## 4.7 SharedActivity

### 用途
代表某一天或某時段的共同行程。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| TripId | uniqueidentifier | 是 | 所屬旅程 |
| TripDayId | uniqueidentifier | 否 | 對應旅程日期 |
| Date | date | 是 | 共同行程日期 |
| Title | nvarchar(200) | 是 | 共同行程標題 |
| Description | nvarchar(max) | 否 | 活動說明 |
| StartTime | time | 否 | 開始時間 |
| EndTime | time | 否 | 結束時間 |
| LocationName | nvarchar(255) | 否 | 地點名稱 |
| Address | nvarchar(500) | 否 | 地址 |
| CreatedByUserId | uniqueidentifier | 是 | 建立者 |
| CreatedAt | datetime2 | 是 | 建立時間 |
| UpdatedAt | datetime2 | 是 | 更新時間 |

### 備註
- 共同行程與個人行程應可共存在同一天
- MVP 階段可先不做太複雜的審核流程
- 建立者可為團主，或已獲授權可管理共同行程的成員

---

## 4.8 SharedActivityParticipant

### 用途
表示哪些成員參與某個共同行程。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| SharedActivityId | uniqueidentifier | 是 | 所屬共同行程 |
| UserId | uniqueidentifier | 是 | 參與者 |
| ParticipationStatus | nvarchar(50) | 否 | joined / invited / declined |
| CreatedAt | datetime2 | 是 | 建立時間 |

### 備註
- MVP 若所有人都預設參加，也可以先簡化
- 若之後要支援部分成員一起玩，這張表很有用

---

## 4.9 Lodging

### 用途
表示旅程中的住宿資訊。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| TripId | uniqueidentifier | 是 | 所屬旅程 |
| Name | nvarchar(200) | 是 | 住宿名稱 |
| Address | nvarchar(500) | 是 | 地址 |
| CheckInDate | date | 是 | 入住日期 |
| CheckOutDate | date | 是 | 退房日期 |
| RoomInfo | nvarchar(200) | 否 | 房間資訊 |
| Notes | nvarchar(max) | 否 | 備註 |
| CreatedAt | datetime2 | 是 | 建立時間 |
| UpdatedAt | datetime2 | 是 | 更新時間 |

### 備註
- MVP 先支援基本住宿紀錄即可
- 先不整合外部住宿平台資料

---

## 4.10 ItineraryCopyLog

### 用途
記錄誰在什麼時候複製了誰的某一天行程，作為後續追蹤與除錯依據。

### 建議欄位

| 欄位 | 類型建議 | 必要 | 說明 |
|---|---|---|---|
| Id | uniqueidentifier / UUID | 是 | 主鍵 |
| TripId | uniqueidentifier | 是 | 所屬旅程 |
| SourceUserId | uniqueidentifier | 是 | 被複製來源的使用者 |
| TargetUserId | uniqueidentifier | 是 | 複製到哪位使用者 |
| SourceDate | date | 是 | 被複製的日期 |
| TargetDate | date | 是 | 貼入的日期 |
| CopiedByUserId | uniqueidentifier | 是 | 執行複製的人 |
| CreatedAt | datetime2 | 是 | 複製時間 |

### 備註
- MVP 不是絕對必要，但很建議保留
- 對除錯、追蹤與未來功能擴充有幫助

---

## 5. 實體關係概念

## 5.1 User 與 Trip
- 一位 User 可以建立多個 Trip
- 一個 Trip 只有一位團主（OwnerUserId）
- 一位 User 也可以作為成員參與多個 Trip

## 5.2 Trip 與 TripMember
- 一個 Trip 有多個 TripMember
- 一個 TripMember 對應一位 User 在某個 Trip 中的角色

## 5.3 Trip 與 PersonalItinerary
- 一個 Trip 可包含多位成員的多份個人行程
- 一位成員在同一旅程中可有多天的個人行程

## 5.4 PersonalItinerary 與 PersonalItineraryItem
- 一份 PersonalItinerary 包含多個 PersonalItineraryItem
- 一個 Item 必須屬於一份 PersonalItinerary

## 5.5 Trip 與 SharedActivity
- 一個 Trip 可包含多個 SharedActivity
- 多個 SharedActivity 可分布在不同日期或時段

## 5.6 SharedActivity 與 SharedActivityParticipant
- 一個 SharedActivity 可有多位參與者
- 一位使用者也可參與多個 SharedActivity

## 5.7 Trip 與 Lodging
- 一個 Trip 可有一筆或多筆住宿資料
- 多筆住宿可支援多城市或多段住宿情境

---

## 6. MVP 必要欄位與可延後欄位

## 6.1 MVP 必要
以下資料建議列為 MVP 必要：

- User：Id、DisplayName、Email
- Trip：Name、StartDate、EndDate、OwnerUserId
- TripMember：TripId、UserId、Role
- PersonalItinerary：TripId、UserId、Date
- PersonalItineraryItem：Title、Category、SortOrder
- SharedActivity：TripId、Date、Title
- Lodging：Name、Address、CheckInDate、CheckOutDate

## 6.2 可延後
以下欄位可於後續再補：

- User：IsActive 以外的更多帳號設定
- Trip：Status
- TripDay：DayNumber
- PersonalItineraryItem：Address、Notes 更細節欄位
- SharedActivityParticipant：ParticipationStatus
- ItineraryCopyLog：若第一版想更快交付，可延後
- Lodging：RoomInfo 以外的更多細節欄位

---

## 7. 建議的資料表分組

若未來轉成 SQL Server 資料表，可先分為以下幾組：

### 7.1 帳號與成員
- Users
- TripMembers

### 7.2 旅程主體
- Trips
- TripDays

### 7.3 個人行程
- PersonalItineraries
- PersonalItineraryItems

### 7.4 共同行程
- SharedActivities
- SharedActivityParticipants

### 7.5 住宿與操作紀錄
- Lodgings
- ItineraryCopyLogs

---

## 8. 待確認事項

以下項目仍需後續確認：

- 是否要保留 `TripDay` 這張表，或直接用日期欄位即可
- 團主是否可直接編輯他人的個人行程
- 共同行程是否一定要指定參與成員
- 行程複製時，目標日期是否一定與來源日期相同
- 住宿資訊是否只允許團主管理，或也可授權一般成員管理
- 是否只用 TripMember.CanManageSharedActivities 控制共同行程權限，或未來擴充更多細部權限

---

## 9. 下一步建議

完成本文件後，建議下一步整理：

- `docs/requirements/api-draft.md`

用來定義：
- 各功能對應的 API 分類
- 端點用途
- 權限範圍
- request / response 草稿
