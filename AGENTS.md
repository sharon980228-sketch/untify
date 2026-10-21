# Universal Unity Project Rules (AGENTS.md)

## 1. Environment & Project Context (請依各專案勾選/設定)
- **Engine Version**: [Unity 6 (6000.x) / Unity 2022 LTS]
- **Dimension**: [3D / 2D] *(若為 2D，物理元件一律使用 2D 系列：Rigidbody2D, Collider2D)*
- **Render Pipeline**: [URP / HDRP / Built-in]
- **Input Handling**: [Unity Input System (New) / Legacy Input Manager]
- **UI Framework**: [UGUI + TMPro / UI Toolkit]
- **Architecture Pattern**: [Event-Driven (Action) / ScriptableObject Events / VContainer / UniRx]
- **Project Game Design Document (GDD)**: 請檢索並遵循當前專案的企劃規格文件目錄：@[專案企劃文件目錄](Docs/)

---

## 2. Core Philosophy: Editor-First & Component-Driven (編輯器優先鐵律)
為了確保專案具備視覺化維護性與符合標準 Unity 開發習慣，Agent 必須嚴格遵守以下原則：
1. **嚴禁「純程式碼刻畫面/刻物件」**：
   - **UI 介面**：嚴禁在 C# 中使用程式碼動態建立 Canvas、Panel、Button、Image 或 TextMeshPro 元件。UI 的佈局、字型、排版與樣式**必須完全在 Unity Editor 內的 Canvas 階層建立**。
   - **實體物件**：嚴禁在遊戲執行階段使用 `GameObject.CreatePrimitive()` 動態手刻模型（如用程式產生球體當子彈）。所有實體必須使用 Prefab。
2. **職責分離（Data-Driven & View-Binding）**：
   - C# 腳本的職責僅限於**「宣告暴露給 Inspector 的參照（`[SerializeField]`）與操作該參照的邏輯」**。
   - 畫面外觀、文字樣式、錨點對齊（Anchor）、碰撞體大小、音效與粒子特效，一律交由開發者在 Unity Editor 與 Prefab 介面中視覺化調整。
3. **Inspector 調校友善（No Hardcoding）**：
   - 所有影響手感與表現的數值（如移動速度、射速、冷卻時間、顏色、UI 動畫時間），一律宣告為具備 `[SerializeField]` 與 `[Tooltip]` 的變數，嚴禁寫死在程式碼內。

---

## 3. Language & Code Documentation Standards (語言與繁體中文註解規範)
- **溝通與交付語言**：所有對話、架構說明、驗證報告與驗收清單，一律使用**繁體中文（台灣習慣用語）**。
- **程式碼繁體中文註解（強制執行）**：
  - **類別與方法註解**：所有 `Class`、`Interface`、`Enum` 以及 `public` / `protected` 方法，必須加上繁體中文的 XML 文件註解（`/// <summary> ... </summary>`），說明其職責、參數意義與回傳值。
  - **核心邏輯行內註解**：在狀態機切換、物理射線檢測、數學向量運算、計時器與事件觸發等關鍵段落，必須加上清晰的繁體中文行內註解（`// ...`），解釋「為什麼這樣寫（Why）」而非僅翻譯語法。
  - **Inspector 提示中文化**：所有 `[SerializeField]` 欄位皆須搭配 `[Tooltip("繁體中文說明")]`，清楚標示該參數在編輯器中的用途與單位（例如：`[Tooltip("子彈飛行速度（單位：公尺/秒）")]`）。

---

## 4. Iterative Workflow & Verification Protocol (分段開發與驗收鐵律)

### 4.1 分段規劃與嚴格停等機制 [CRITICAL GATEKEEPING RULE]
1. **禁止一次生成全部系統**：收到大型需求或企劃書時，必須先將開發任務拆解為多個獨立且可測試的「開發階段（Milestones / Phases）」，並先列出階段規劃供開發者審閱。
2. **單一階段執行原則**：每次僅允許實作「當前單一階段」的程式碼。
3. **強制停等（HARD STOP）**：
   - 當一個階段的程式碼撰寫與 AI 自動驗證完成後，**必須立即停止動作**。
   - **在開發者明確回覆「確認」、「測試通過」或「進入下一階段」之前，嚴禁擅自修改現有程式碼，也嚴禁主動開始撰寫下一階段的功能**。
   - 若開發者回報測試問題，僅針對該階段的問題進行修復與重新驗證，修復後同樣必須等待開發者確認。
   - 階段交付結尾必須附帶固定標記：`🛑 [MILESTONE PAUSED] 等待測試確認`。

### 4.2 AI 自動驗證清單 (AI Pre-Delivery Self-Verification)
在將任何程式碼交付給開發者之前，AI 必須先執行內部自動驗證（若有連接 Unity MCP / 終端機則讀取編譯結果，否則進行靜態邏輯審查），並在回覆中附上**「AI 自動驗證報告」**，確認以下項目皆已通過：
- [ ] **語法與編譯檢查**：確認所有命名空間（Namespace）、型別引用、方法簽章正確無誤，無遺漏分號或括號。
- [ ] **版本 API 相容性**：Unity 2022+ / Unity 6 避免使用已棄用 API（如以 `FindFirstObjectByType<T>()` 取代 `FindObjectOfType<T>()`）。
- [ ] **Null Safety 防呆**：確認所有外部參照、`GetComponent` 與事件呼叫（`?.Invoke`）皆有防呆保護，避免在 Play Mode 產生 `NullReferenceException`。
- [ ] **記憶體與生命週期檢查**：確認所有 `OnEnable` 訂閱的事件皆有在 `OnDisable` 或 `OnDestroy` 中取消訂閱；確認 `Update` 迴圈中無高耗能查找 API。
- [ ] **視覺化規範查核**：確認無直接以代碼手刻 UI 或 Procedural 幾何體的違規寫法。

### 4.3 開發者手動驗收標準 (Developer Manual Acceptance Criteria)
每次完成單一階段交付時，AI 必須在回覆結尾提供結構化的**「Unity Editor 設定與手動驗收清單」**，格式如下：
1. **Unity Editor 視覺化設定指引**：
   - 明確說明需在 Hierarchy 新增的物件結構（例如：`Canvas -> HUD -> AmmoText (TextMeshPro)`）。
   - 明確說明需手動建立的 Prefab（例如：建立 Sphere -> 掛載腳本 -> 存為 `Bullet.prefab`）。
   - 列出 Inspector 上所有 `[SerializeField]` 欄位應如何拖曳綁定。
2. **手動測試驗收項目（Checklist）**：列出具體的「操作動作 → 預期結果」供開發者在 Play Mode 逐項測試。
3. **停等確認詢問**：在結尾明確詢問：「請在 Unity 中依照上述驗收標準進行測試，確認無誤後請回覆『確認通過』，我才會進入下一階段開發。」

---

## 5. Directory Structure & Naming Conventions
- **資產隔離**：所有專案自製內容一律放在 `Assets/_Project/` 下：
  - `Assets/_Project/Scripts/`（依模組建立子資料夾：`Core`, `Player`, `AI`, `Gameplay`, `UI`, `Utilities`）
  - `Assets/_Project/Prefabs/`、`Assets/_Project/ScriptableObjects/`、`Assets/_Project/Scenes/`
- **C# 命名標準**：
  - `Class` / `Struct` / `Interface` / `Enum` / `Method` / `Property` / `Event`：使用 `PascalCase`（介面以 `I` 開頭）。
  - `Private` / `[SerializeField] private` 欄位：使用 `_camelCase`，嚴禁使用 `public` 欄位暴露 Inspector 參數。
  - 每個 `.cs` 檔案僅能包含一個主要型別，且檔名必須與型別名稱完全一致。

---

## 6. Architecture, Performance & Asset Guardrails
- **單一職責與事件驅動**：拆分肥大的 `MonoBehaviour`；遊戲邏輯與 UI 必須透過 `event System.Action<T>` 解耦，嚴禁在 UI 的 `Update()` 中每幀輪詢狀態。
- **效能禁令**：嚴禁在 `Update()`、`FixedUpdate()`、`LateUpdate()` 中呼叫搜尋 API（如 `Find`、`GetComponent`）；所有元件參照必須在 `Awake()` 中快取；標籤比對一律使用 `CompareTag`。
- **資產與中繼資料（.meta）守則**：
  - 除非透過已啟用的 Unity MCP 工具操作，否則**絕對禁止**以純文字方式直接建立、修改或刪除 `.unity`、`.prefab`、`.asset` 等序列化檔案內容。
  - 當需要新增 Prefab 或 Scene 時，AI **必須提供 C# 腳本 + 編輯器手動建立步驟**，嚴禁以「無 MCP 工具」為由拒絕或中止任務。
  - 新增 C# 腳本時僅建立 `.cs` 檔案，由 Unity 編輯器自動產生 `.cs.meta`；若刪除或移動 `.cs` 檔案，必須同步清除舊位置的 `.meta`。
  - 未經指示不得任意新增或修改 `.asmdef`（Assembly Definition）檔案。