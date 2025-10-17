# 需求矩陣（能力對照 / 取捨 / 在地化清單）

| 面向 | 能力 | Community | Enterprise | 在地自研 Addons | 補習班優先度 | 備註 |
|---|---|---|---|---|---|---|
| 課時/課表/預約 | 課程/班級/學生/課表/出缺勤/補課 | Project+Timesheet+Planning+Appointment 可拼 | 無教育垂直 | 教務模型+流程（需） | 高 | 你的核心差異化 |
| 收費/金流 | 線上繳費、對賬 | Payment Framework，有少量 Provider | 仍需第三方 API | 台灣支付 Provider（綠界/藍新/TapPay）（需） | 高 | 先上一家後擴充 |
| 電子簽章 | 前台簽名、PDF 壓印、稽核 | 無 | sign 完整 | e‑sign MVP（已出技術文檔）（需） | 中 | 合規等高階可後續擴充 |
| 文件/知識 | 歸檔、知識庫 | 限制 | documents/knowledge | 輕量歸檔（附件+標籤）（選） | 中 | 可先簡化 |
| 進銷存/採購 | 基本入出庫/採購 | 有（基礎） | 強（條碼/物流/成本） | 精簡倉儲/報表（選） | 低 | 補習班不優先 |
| 會計/報表 | 總帳/報表 | 有（基礎） | 強（報表/自動化） | 台灣報表/發票（選） | 中 | 依現場需求 |
| HR/薪資 | 工時/排班/請假/薪資 | 有（基礎） | 多國但無台灣 | 台灣薪資規則（需） | 中高 | 取決於鐘點/結算深度 |
| UI/首頁 | App Grid、搜尋 | 有（可自製） | web_enterprise | 已有 `web_home_enterprise_like`/`apps_dashboard` | 中 | 可沿用 |

決策建議
- 補習班/家教：Community + 你方 Addons 足以切題，Enterprise 非必須
- 傳統買賣（簡單流程）：Community + 精簡倉儲/報表足以；若要條碼/物流/跨國帳務再評估 Enterprise
