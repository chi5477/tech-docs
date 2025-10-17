# Odoo 使用者介面精簡與權限配置指南（無示範資料）

目的：降低一般使用者看到的功能數量，避免混淆，同時保留必要的工作流程與資料安全。本文提供「不用寫程式」的操作做法，以及「可版控」的進階模組做法。

---

## 核心原則
- 先少後多：只安裝需要的 App，其他一律不安裝。
- 以權限驅動可見性：不給權限 = 不顯示選單與 App 方塊。
- 隱藏不等於安全：若真的不讓用，必須同時移除模型（Model）的存取權與記錄規則（Record Rules）。
- 以角色設計權限：先定義「角色/職務 → 能做什麼」，再對應到 Odoo 群組（Group）。
- 預設首頁清爽：將所有人登入導向「主選單（Home Menu）」或指定唯一 App。

---

## 快速開始（十分鐘完成）
1) 建新資料庫且不含示範資料
- 透過資料庫管理介面：建立新 DB 時「取消勾選」載入示範資料。
- 或用命令：`odoo-bin -d <db> --without-demo=all -i base`

2) 僅安裝必需 App
- 先從最小集合開始（例：Contacts、Sales 或你實際要用的單一 App）。

3) 設定使用者類型與基本權限
- 一般員工用「內部使用者」，但不要給「設定（Administration: Settings）」權限。
- 不需要的 App 權限一律不要勾選。

4) 設定登入首頁
- 每位使用者的「首頁動作（Home Action）」設為「主選單（Home Menu）」；網址亦可用 `/web#home`。

5) 建測試帳號驗證
- 用無痕視窗登入測試帳號，畫面只應出現被授權的 App 方塊與選單。

---

## 建立乾淨資料庫（無示範資料）
- 設定檔方式：`odoo.conf` 加入
```
[options]
without_demo = all
```
- 以設定檔啟動：`odoo-bin -c C:\path\to\odoo.conf`
- 之後安裝任何模組時，若用命令也維持 `--without-demo=all`，避免意外載入 demo。

---

## 權限與可見性策略

### 1. 使用者類型
- 內部使用者（Internal）：能登入後台，適合需要處理單據的員工。
- 入口網站使用者（Portal）：僅可使用前台入口與被分享的紀錄，極簡但不能進入後台主選單。

### 2. 權限群組規劃（建議）
- 依職務建立 2–4 個分層群組，例如：
  - `員工-基礎`：唯讀聯絡人、能看討論（可選）。
  - `銷售-一般`：在「員工-基礎」之上，新增銷售/報價權限。
  - `經理-銷售`：在「銷售-一般」之上，新增審批/報表權限。
- 一般員工不要賦予「設定（Settings）」與「應用程式（Apps）」相關權限，避免自行安裝。

### 3. 只顯示必要 App 的做法
- 規則：Home Menu 上的 App 方塊來自頂層選單與其關聯的群組。使用者沒有對應群組，就看不到該 App。
- 實作：
  1) 設定 > 使用者與公司 > 使用者：只勾選他需要的 App 權限（例如只給「銷售使用者」）。
  2) 設定 > 技術 > 使用者介面 > 選單：對個別選單設定「群組」限制，讓它只對特定群組可見（需開發者模式）。
  3) 如需更嚴格，進一步在「設定 > 技術 > 安全性 > 存取控制與記錄規則」移除不需模型的 `read/write/create/unlink` 權限。

### 4. 隱藏「應用程式」與「設定」
- 不要給一般使用者「Administration: Settings」或任何管理群組，則上方的「設定」選單與「應用程式」入口自然消失。

### 5. 首頁與導向
- 使用者偏好中的「首頁動作」改為「主選單（Home Menu）」；或把分享給使用者的後台連結統一為 `/web#home`。

---

## 僅用 UI 的精簡步驟（無需寫程式）
1) 啟用開發者模式。
2) 到「應用程式」只安裝必要 App。
3) 到「設定 > 使用者與公司 > 使用者」：對每位使用者
   - 使用者類型：內部使用者。
   - 僅勾選必要 App 權限；絕不給 Settings 權限。
   - 首頁動作：主選單（Home Menu）。
4) 到「設定 > 技術 > 使用者介面 > 選單」：
   - 找到不想給一般人看的選單，設定「群組」欄位為特定群組（例如只給經理）。
   - 注意：這會影響所有使用者；請用群組精準圈選。
5) 驗證：用無痕視窗登入測試帳號，檢查 Home Menu 是否只剩必要 App。

---

## 用 Odoo Studio 的精簡（可選）
- 可在 Studio 中對表單/清單的欄位、按鈕設定「可見群組」，讓一般人畫面更簡潔。
- Studio 也可快速新增群組並綁定到視圖或選單；但仍應同步檢查模型 ACL 與記錄規則，確保安全性。

---

## 以自訂模組實作（推薦給需版控的環境）
當你要把精簡規則版本化、可在多台環境重複部署時，建立小型模組最穩定。

### 範例模組骨架
```
simplified_ui/
├─ __manifest__.py
├─ security/
│  ├─ ir.model.access.csv   # 需要時才加，預設可留空
│  └─ security_groups.xml   # 建立自訂群組（如 員工-基礎）
└─ data/
   └─ menu_restrict.xml     # 對選單設定群組可見性
```

`__manifest__.py`
```
{
    'name': 'Simplified UI',
    'version': '1.0',
    'depends': ['base'],          # 依實際需要加上 sales、contacts…
    'data': [
        'security/security_groups.xml',
        'data/menu_restrict.xml',
    ],
    'installable': True,
}
```

`security/security_groups.xml`
```
<odoo>
  <record id="group_employee_basic" model="res.groups">
    <field name="name">員工-基礎</field>
    <field name="category_id" ref="base.module_category_hidden"/>
  </record>
</odoo>
```

`data/menu_restrict.xml`（示例：把某些頂層選單限制為經理群組可見）
```
<odoo>
  <!-- 將選單可見性改為只對特定群組開放；實際外部 ID 請於「開發者模式 > 技術 > 選單」查詢後替換 -->
  <record id="sale_menu_root_patch" model="ir.ui.menu">
    <field name="groups_id" eval="[(6,0,[ref('sales_team.group_sale_manager')])]"/>
  </record>
</odoo>
```
說明：
- 以相同 external ID（如 `sale.menu_sale_root`）建立 `<record>` 可更新該選單的群組。請先到系統查到確切的 external ID 再填入。
- 若選單由原模組以 `noupdate="1"` 建立，仍可由你的模組再次寫入更新；若遇到無法更新，可改以伺服器動作（post-init hook）寫入，或直接限制其對應的動作（`ir.actions.*`）群組。

> 注意：僅限制選單不等於移除後端資料存取。如需完全禁止，還要移除對應模型的 ACL、Record Rules。

---

## 角色範本（示意）
- 銷售人員：`員工-基礎` + `銷售使用者`（不含設定權限）。
- 銷售經理：`員工-基礎` + `銷售經理`。
- 只看聯絡人：`員工-基礎`（聯絡人唯讀 ACL）。
- 客戶（Portal）：僅 `Portal` 群組，靠分享文件工作。

---

## 驗收清單
- [ ] 測試帳號登入只看到預期 App 方塊。
- [ ] 上方無「設定」與「應用程式」。
- [ ] 未授權的功能在網址列直接輸入也會被拒絕（ACL/Rules 生效）。
- [ ] 首頁進入 `/web#home`。

---

## 常見問題與風險
- 只藏選單無法阻擋 API 或直接 URL：必須同步調整 ACL 與記錄規則。
- Studio 調整要記得「對群組」做可見條件；否則所有人都會看到。
- 給了「Settings」或管理群組的使用者，仍能看到 Apps 與更多功能。
- 安裝新 App 後，可能新增選單與群組，請記得更新你的角色與群組設計。

---

## 建議維運流程
1) 先在測試資料庫設計與驗證權限與可見性。
2) 將結果轉為自訂模組（可選）以便版本控制與環境複製。
3) 發布到正式機前，使用測試帳號逐項驗收清單。
4) 每次安裝新 App 或升級版本後，重新審視選單與權限。

---

## 參考操作（PowerShell）
- 建立乾淨 DB：
```
.\odoo-bin -d mydb --without-demo=all -i base
```
- 安裝必要 App（例：contacts, sale），仍不載入 demo：
```
.\odoo-bin -d mydb -i contacts,sale --without-demo=all
```
- 以設定檔啟動：
```
.\odoo-bin -c C:\path\to\odoo.conf
```

---

如需我幫你出一個可直接安裝的「Simplified UI」模組，把要保留的 App 與角色告訴我，我可以依此產出模組與部署步驟。
