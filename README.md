# 🎁 聖誕交換禮物抽籤網站

一個完全免費的交換禮物抽籤網站：大家在網頁上報名，時間到由主辦人按下抽籤，每個人再回到同一個網站查看自己要送禮的對象。

- 前端：純 HTML + JavaScript，部署在 **GitHub Pages**
- 後端：**Supabase**（Postgres 資料庫與函式），免費方案即可
- 不需要自己架伺服器，也不需要寫後端程式

## 功能

- 報名：輸入名字並自訂密碼
- 抽籤：主辦人輸入密碼後執行，只能抽一次
- 排除配對：指定誰不能抽到誰（例如情侶），雙向生效
- 查詢：輸入名字與密碼，只能看到自己的送禮對象
- 人數不足或排除條件太多導致無解時，會顯示提示

## 使用方式

### 參加者
1. 打開網站，在「報名」頁輸入名字與密碼（至少 4 碼，建議 6 碼以上）
2. 等主辦人通知抽籤完成
3. 在「查詢對象」頁輸入名字與密碼，查看自己要送給誰，並保密到交換禮物當天

### 主辦人
1. 在「主辦人」頁輸入主辦人密碼，按「載入報名名單」檢查有沒有陌生名字
2. 在文字框輸入不能互抽的配對，每行一組，用逗號分隔，例如：
   ```
   小明,小美
   阿強,阿花
   ```
3. 按「抽籤」並確認。至少需要 3 人，抽完後不能重抽，也不能再報名
4. 通知大家回網站查詢

## 專案檔案

| 檔案 | 說明 |
|---|---|
| `index.html` | 整個網站（報名、查詢、主辦人三個分頁） |
| `schema.sql` | Supabase 的資料表、權限與函式 |
| `README.md` | 本說明 |

## 部署步驟

### 1. 建立 Supabase 專案
1. 到 [supabase.com](https://supabase.com) 註冊並建立免費專案，Region 建議選離台灣近的（東京或新加坡）
2. 建立專案時保持勾選 **Enable Data API**
3. 等專案建立完成（約 1～2 分鐘）

### 2. 建立資料表與函式
1. 打開 `schema.sql`，把 `CHANGE_ME` 換成你的主辦人密碼（建議 12 字以上）
2. 到 Supabase 的 **SQL Editor**，貼上整份內容並執行，看到 Success 即完成

### 3. 填入連線資料
1. 到 **Project Settings → API**，複製 Project URL 與 publishable（或 anon）key
2. 填進 `index.html` 最上方：
   ```js
   const SB_URL = 'https://你的專案.supabase.co';
   const SB_KEY = '你的 publishable key';
   ```
   請不要使用 secret key 或 service_role key

### 4. 發布到 GitHub Pages
1. 建立 **Public** 的 GitHub repo，把 `index.html` 放在最外層
2. 到 **Settings → Pages**，Source 選 Deploy from a branch，Branch 選 `main`、資料夾選 `/ (root)`，按 Save
3. 等 1～3 分鐘，取得網址 `https://你的帳號.github.io/repo名稱/`

> 若要把 `schema.sql` 一起放上 GitHub，請保持裡面的 `CHANGE_ME` 原樣，不要提交真正的主辦人密碼。

### 5. 測試
先用 3 個假名字完整跑一次報名、抽籤、查詢，確認無誤後，執行下面的「重設」SQL 清空測試資料，再正式開放報名。

## 重設與維護

在 Supabase 的 SQL Editor 執行。

**只重設抽籤（保留報名名單，想重抽時用）**
```sql
update settings set value = 'false' where key = 'drawn';
delete from assignments where giver is not null;
delete from exclusions where a is not null;
```

**全部清空（測試完、正式開放前用）**
```sql
update settings set value = 'false' where key = 'drawn';
delete from participants where id is not null;
```

**更換主辦人密碼**
```sql
update settings set value = extensions.crypt('新密碼', extensions.gen_salt('bf')) where key = 'admin_hash';
```

> 正式活動抽籤後請不要再重抽，否則大家看到的對象會跟之前不同。

## 安全性說明

- `SB_URL` 與 publishable key 設計上就是給瀏覽器使用的，公開在網頁裡是正常的
- 所有資料表都已開啟 Row Level Security 並撤銷直接存取權限，前端只能呼叫指定函式
- 查詢結果需要「名字＋密碼」，且一次只回傳該人自己的對象
- 密碼與主辦人密碼都以雜湊（bcrypt）儲存，不存明文
- 擁有 Supabase 後台權限的人（也就是你）看得到 `assignments` 資料表；想保持驚喜就不要打開它

**不可公開的資料**：secret key / service_role key、資料庫密碼、主辦人密碼。

### 已知限制
- 知道網址的人可以替陌生名字報名，抽籤前請先檢查名單
- 密碼太短可能被猜測，請大家設 6 碼以上
- 沒有登入機制，忘記密碼只能由主辦人在後台處理

## 常見問題

| 狀況 | 處理方式 |
|---|---|
| 顯示「連線失敗」並附上錯誤訊息 | 依錯誤訊息排查；常見原因是 `schema.sql` 沒有完整執行，或網址、金鑰填錯 |
| 顯示「已經抽過籤了」 | 這次活動已抽過；要重測請用上面的「只重設抽籤」SQL |
| 顯示「排除條件太多，找不到可行的配對」 | 減少排除組合後再抽 |
| 打開網址看到 404 | 確認檔名是 `index.html` 且在最外層，並等 Pages 部署完成 |
| 載入名單顯示失敗 | 通常是主辦人密碼不正確 |
| Supabase 專案被暫停 | 免費專案閒置一段時間可能暫停，活動前登入 Supabase 確認狀態 |

## 授權

自由使用與修改。
