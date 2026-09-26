# extract-official-guide - 技術文件

> 返回 [README](./README.zh.md)

## 前置需求

- 可載入 skill 的 agent 執行環境（本文以 `~/.claude/skills/` 為安裝位置）
- 可連線至 `platform.claude.com`、`developers.openai.com`、`ai.google.dev` 與 `web.archive.org`
- 產出目標 repo 內存在 `configs/prompts/system_prompt/official_guides/`（預設為 Agenvoy；其他 repo 須以 `--to` 指定）
- Go 工具鏈（Verify 階段執行 `go build -tags fts5 ./...`）
- Python 3（Verify 階段的標題順序檢查）

## 安裝

### 從原始碼安裝

```bash
git clone https://github.com/agenvoy/skill-extract-official-guide.git ~/.claude/skills/extract-official-guide
```

### 確認載入

```bash
ls ~/.claude/skills/extract-official-guide/SKILL.md
```

重新載入 skill 後，清單會出現 `extract-official-guide`。

## 設定

本 skill 不讀取環境變數，所有行為由指令參數與下列路徑決定。

| 項目 | 路徑 | 說明 |
|------|------|------|
| 來源留底 | `~/.claude/skills/extract-official-guide/offical_guide/<key>.md` | 廠商原始全文，首行為 `Retrieved <YYYY-MM-DD> from <url>` |
| 預設產出目錄 | `<repo>/configs/prompts/system_prompt/official_guides/` | 目錄不存在時會反問，不自動建立 |

## 使用方式

### 基礎用法

對來源登錄表中的全部 key 依序執行下載、萃 base、萃模型檔：

```text
/extract-official-guide
```

### 指定型號

只處理指定的 key；給了 key 時預設跳過階段 B，不動一律注入的 base 檔：

```text
/extract-official-guide claude-opus-5-5 gpt-5.6
```

### 只下載或只萃取

```text
# 只更新留底，不萃取
/extract-official-guide gemini --download-only

# 跳過下載，直接以本地留底萃取
/extract-official-guide gpt-5.4 --extract-only
```

### 進階用法

```text
# 指定其他 repo 的產出目錄
/extract-official-guide claude-sonnet-5 --to ~/Desktop/git/other-repo/prompts/official_guides

# 只重萃 _base.md 與 _base_unlisted.md
/extract-official-guide --base-only
```

留底或產出檔已存在時，skill 會先輸出差異摘要並詢問是否覆寫；未取得同意不會寫入。

## 命令列參考

### 指令

| 指令 | 語法 | 說明 |
|------|------|------|
| `/extract-official-guide` | `/extract-official-guide [<key>...] [--to <DEST>] [--download-only] [--extract-only] [--base-only] [--skip-base]` | 下載官方 prompting guide 並萃取為模型指引檔 |

### 旗標

| 旗標 | 說明 |
|------|------|
| `<key>...` | 只處理指定 key（例：`claude-opus-5`、`gpt-5.4`、`gemini`）；未給則處理全部 |
| `--to <DEST>` | 指定產出目錄；未給則使用預設目錄 |
| `--download-only` | 完成下載後停止，不萃取 |
| `--extract-only` | 跳過下載，使用本地留底萃取 |
| `--base-only` | 只跑階段 B，不動模型檔 |
| `--skip-base` | 跳過階段 B；給了 `<key>` 時為預設 |

### 來源登錄表

| 廠商 | key | 來源 |
|------|-----|------|
| Anthropic | `claude`（總表） | `platform.claude.com/.../claude-prompting-best-practices.md` |
| Anthropic | `claude-<model>` | 同路徑下 `prompting-claude-<model>.md`，URL 取自總表的 model-specific guidance 表 |
| OpenAI | `gpt-<ver>` | `developers.openai.com/api/docs/guides/latest-model/gpt-<ver>.md` |
| Google | `gemini` | `ai.google.dev/gemini-api/docs/prompting-strategies` |

舊型號 URL 被新家族頁取代時改走 Wayback，並將 snapshot timestamp 寫入留底首行。

### 產出檔案

| 檔案 | 上限 | 注入時機 |
|------|------|----------|
| `_base.md` | 25 行 / 4 區塊 | 一律 |
| `_vendor_<vendor>.md` | 35 行 / 5 區塊 | 模型名含該廠商字樣時 |
| `<key>.md` | 55 行 / 6 區塊 | 型號命中時（長 key 優先，僅命中一份） |
| `_base_unlisted.md` | 25 行 / 4 區塊 | 型號檔與廠商檔皆未命中時 |

### 標題字彙

依序只能使用：`Acting`、`Scope`、`Instructions`、`Tools`、`Delegation`、`Grounding`、`Verification`、`Review`、`Long context`、`Long horizon`、`Progress`、`Output`、`Code`、`Other`。

### 刪除理由

| 理由 | 判準 |
|------|------|
| 與 `_base.md` 重複 | 逐字或近逐字同一條規則；帶 nuance 者保留 |
| 與共通層重複或相反 | 以 `system_prompt.md` §Behavioral Constraints 為準 |
| 與本地契約相反 | 本地契約優先 |
| 檔案內自相矛盾 | 保留較具體的一條 |
| 思考指導 | 「想多深、多用心」類句子 |
| 流程步驟 | 逐步 SOP |
| 怎麼用某個工具 | 移至該工具的 description |
| prompt 撰寫建議 | 教開發者寫 prompt，而非 steer 模型 |
| 具名案例 | 具名字型、公司、版本號 |

### 已留底的 key

| 廠商 | key |
|------|-----|
| Anthropic | `claude`、`claude-fable-5`、`claude-fable-5-1`、`claude-opus-4-8`、`claude-opus-5`、`claude-opus-5-5`、`claude-sonnet-5` |
| OpenAI | `gpt-4.1`、`gpt-5`、`gpt-5.1`、`gpt-5.2`、`gpt-5.3`、`gpt-5.4`、`gpt-5.5`、`gpt-5.6`、`gpt-6`、`gpt-6-astra` |
| Google | `gemini` |
