# extract-offical-guide - 架構

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    V[廠商官網<br>Anthropic / OpenAI / Google] --> A[階段 A：下載]
    A --> O[(offical_guide/ 留底)]
    O --> B[階段 B：萃 base]
    B --> BF["_base / _base_unlisted / _vendor_*"]
    O --> C[階段 C：萃模型檔]
    BF -->|去重對照| C
    C --> MF["&lt;key&gt;.md"]
    BF --> D[階段 D：驗證與報告]
    MF --> D
    D --> I[officialGuideSection 分層注入]
```

## Module: 階段 A 下載

將 key 解析為官方 URL，抓取 markdown 全文並與既有留底比對。

```mermaid
graph TB
    subgraph 階段A[階段 A：下載]
        R[A1 解析<br>key → URL] --> F[A2 抓取全文]
        F --> Q{留底已存在？}
        Q -->|否| W[寫入留底<br>首行 Retrieved 日期與 URL]
        Q -->|是| DF[A3 diff 比對]
        DF -->|無差異或僅日期行| S[不寫入]
        DF -->|內容差異| ASK[摘要變動區塊並詢問]
        ASK -->|同意| W
        ASK -->|拒絕| S
    end
    REG[來源登錄表] --> R
    R -->|未登錄| U[反問使用者 URL]
    F -->|live URL 被取代| WB[Wayback snapshot]
    WB --> Q
    F -->|抓取失敗| FAIL[報告失敗<br>保留舊留底]
```

## Module: 階段 B 萃 base

依不同來源產出一律注入的結構規則、未列名模型基線與廠商跨型號層。

```mermaid
graph TB
    subgraph 階段B[階段 B：萃 base]
        B1[B1 結構性規則] --> BASE["_base.md<br>Instruction conflicts / Tools"]
        B2[B2 跨廠商共通點] --> UNL["_base_unlisted.md<br>Acting / Long inputs / Verification / Scope"]
        B2b[B2b 廠商總表] --> VEN["_vendor_&lt;vendor&gt;.md"]
        BASE --> B3{B3 重疊檢查<br>同類規則 ≥ 3 份模型檔？}
        B3 -->|是| UNL
        B3 -->|否| B4[B4 落檔<br>既存則 diff 並詢問]
        UNL --> B4
        VEN --> B4
    end
    LOCAL[harness 本地契約] --> B1
    ARCH[(offical_guide/ 留底)] --> B2
    ARCH --> B2b
    PREV[上一版模型檔] --> B3
```

## Module: 階段 C 萃模型檔

逐 key 將留底全文刪減為型號專屬的反預設修正。

```mermaid
graph TB
    subgraph 階段C[階段 C：萃模型檔]
        C1["C1 盤點 ## 區塊"] --> C2[C2 區塊篩選<br>一律不收清單]
        C2 --> C3[C3 條目篩選<br>刪除理由表]
        C3 --> C4[C4 去重<br>只扣 _base.md]
        C4 --> C5[C5 對照共通層]
        C5 --> C6[C6 落檔<br>既存則 diff 並詢問]
    end
    ARCH[(offical_guide/ 留底)] --> C1
    BASE["_base.md"] --> C4
    SP["system_prompt.md<br>Behavioral Constraints"] --> C5
    GUIDE["configs/prompts/guide/*.md"] --> C5
    TOOLS[工具 Description] --> C5
    C6 --> OUT["&lt;DEST&gt;/&lt;key&gt;.md"]
```

## Module: 階段 D 驗證

寫入後實跑篇幅、字彙、順序、重疊、build 與注入分岔檢查。

```mermaid
graph LR
    subgraph 階段D[階段 D：驗證]
        V1[行數與區塊數] --> V2[禁用區塊殘留]
        V2 --> V3[空區塊]
        V3 --> V4[_base 重疊計數]
        V4 --> V5[標題字彙]
        V5 --> V6[標題順序與 Other 條數]
        V6 --> V7[省略號與全形字元]
        V7 --> V8[go build 與 grep -a 嵌入]
        V8 --> V9[注入分岔實測]
    end
    V9 --> RPT[輸出報告]
```

## Module: 分層注入

`officialGuideSection()` 每層各挑一份疊加注入。

```mermaid
graph TB
    M[模型名] --> N[claude 開頭時<br>. 正規化為 -]
    N --> K{型號檔命中？<br>長 key 優先}
    N --> VQ{含廠商字樣？}
    BASE["_base.md"] --> P[注入內容]
    K -->|是| MF["&lt;key&gt;.md"] --> P
    VQ -->|是| VF["_vendor_&lt;vendor&gt;.md"] --> P
    K -->|否| BOTH{兩者皆未命中？}
    VQ -->|否| BOTH
    BOTH -->|是| UNL["_base_unlisted.md"] --> P
```

## 資料流

```mermaid
sequenceDiagram
    participant U as 使用者
    participant S as Skill
    participant W as 廠商官網
    participant A as offical_guide/
    participant D as DEST
    U->>S: /extract-offical-guide [key...]
    S->>W: 抓取 .md 全文
    W-->>S: markdown
    S->>A: diff 既有留底
    A-->>S: 差異摘要
    S->>U: 詢問是否更新留底
    U-->>S: 同意
    S->>A: 寫入留底
    S->>A: 讀取全文
    S->>D: 讀取 _base.md 與既有產出
    S->>U: 輸出產出 diff 並詢問
    U-->>S: 同意
    S->>D: 寫入 base／vendor／模型檔
    S->>D: 執行驗證
    S-->>U: 下載、產出、刪除依據、驗證報告
```

## 狀態機

留底與產出檔共用的覆寫閘門。

```mermaid
stateDiagram-v2
    [*] --> 產生候選
    產生候選 --> 直接寫入: 目標不存在
    產生候選 --> 比對: 目標已存在
    比對 --> 略過: 無差異或僅日期行
    比對 --> 待確認: 內容差異
    待確認 --> 直接寫入: 使用者同意
    待確認 --> 僅報告: 使用者拒絕
    直接寫入 --> 驗證
    略過 --> [*]
    僅報告 --> [*]
    驗證 --> [*]
```
