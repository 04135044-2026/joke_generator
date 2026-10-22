# 中文多輪笑話生成

使用 Ollama 的 Gemma 4 在終端機進行繁體中文多輪對話，將本地笑話素材改編成有情境、人物對話與結尾反轉的完整故事，並支援笑點解釋與情節延伸。

## 語料來源

- 原始素材：[data/corpus.txt](data/corpus.txt)，收錄中文短句，使用 UTF-8 編碼。
- 結構化語料：[data/jokes.json](data/jokes.json)，包含素材編號、主題標籤、原文、笑點說明與延伸內容。
- 笑點說明與延伸內容為整理時補充，供模型回答追問時參考。

### 語料欄位

| 欄位 | 用途 |
| --- | --- |
| `id` | 素材的唯一編號 |
| `tags` | 主題關鍵字，用於挑選相關素材 |
| `text` | 原始笑話短句 |
| `explanation` | 笑點說明，供使用者追問時參考 |
| `follow_up` | 延伸素材，供接續情節時參考 |

執行時讀取 `data/jokes.json`；`data/corpus.txt` 保留原始素材供查閱。

## 基礎模型

- [Ollama：gemma4:e2b](https://ollama.com/library/gemma4:e2b)。
- 透過 Ollama Python 套件呼叫本機模型，使用串流方式顯示回答。
- 目前使用既有模型搭配關鍵字選材與提示詞生成，未進行模型微調。

## 安裝套件

### 主要套件

建議使用 Python 3.12。在專案根目錄以 Windows PowerShell 執行：

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

指令直接使用虛擬環境中的 Python，不需要另外啟用環境腳本。

### 套件版本設定

以下依 [requirements.txt](requirements.txt) 的設定列出，實際安裝版本可用 `.\.venv\Scripts\python.exe -m pip list` 查看。

| 套件 | 安裝設定 |
| --- | --- |
| ollama | `>=0.6,<1` |
| python-dotenv | `>=1.0,<2` |

### 執行環境

- Python 3.12；本機使用 Python 3.12.10 執行。
- Windows PowerShell。
- 安裝並啟動 [Ollama](https://ollama.com/download)，下載 `gemma4:e2b` 模型。
- 模型載入與運算由 Ollama 處理，CPU／GPU 的使用依本機環境決定。

## 說明

程式依使用者輸入選取一則主素材，再依共同主題搭配最多兩則相關素材，交由模型改編成完整笑話。故事會以素材中的雙關或反差作為核心，加入人物、場景、對話及鋪陳。

要求「換一個」或「好笑一點」時，會優先改選未使用的主素材；素材用完後循環選用，並避免連續使用同一則主素材。新故事使用本輪挑選的素材，避免受到上一個故事的情節影響。

詢問「笑點在哪」或「然後呢」時，模型會參照上一個實際生成的故事、對應素材與最近的對話回答。只有使用者要求解釋時才補充笑點說明，程式不設定機器人角色名稱。

### 生成參數

| 項目 | 設定 |
| --- | --- |
| 預設模型 | `gemma4:e2b` |
| 語言 | 繁體中文 |
| 每次選取素材 | 一則主素材，最多兩則搭配素材 |
| 故事長度 | 提示模型生成約 180～300 字、4～6 個短段落 |
| 每次最大生成長度 | 512 tokens |
| 上下文視窗 | 8192 tokens |
| 新故事 temperature | `0.6` |
| 追問 temperature | `0.3` |
| 思考輸出 | 關閉，`think=False` |
| 追問的歷史範圍 | 最近 8 個完整來回 |

故事長度與文風透過提示詞引導，實際內容由模型生成。

### 對話紀錄

每輪回答完成後，使用者輸入與模型回覆會儲存到 `.runtime/hg_chat_history.json`，下次啟動會讀取並接續對話。紀錄包含 `role`、`content`，有使用素材的回覆另保存主素材 `joke_id` 與參考素材清單 `joke_ids`。

`.runtime/` 會在第一次儲存時自動建立。產生失敗或中途中止的回覆不會加入紀錄；輸入 `clear` 會清空紀錄及素材使用狀態。

## 檔案結構

正式專案檔案如下：

```text
專案目錄/
├── app.py                       # 終端機入口、素材選取、模型生成與對話保存
├── data/
│   ├── corpus.txt               # 原始笑話短句
│   └── jokes.json               # 結構化素材、笑點說明及延伸內容
├── requirements.txt             # Python 套件與版本限制
├── .env.example                 # 環境變數範例
├── .gitignore                   # Git 排除清單
└── README.md                    # 專案說明
```

`.venv/`、`.runtime/`、`.env` 與 Python 快取為本機檔案，不納入 Git。
`.runtime/` 在儲存對話時自動建立，不需要預先建立或上傳。

## 使用方式

### 啟動對話

先啟動 Ollama，再於專案根目錄執行：

```powershell
ollama pull gemma4:e2b
.\.venv\Scripts\python.exe app.py
```

模型下載一次後可重複使用，首次載入可能較久。可嘗試輸入「講個人生努力的完整笑話」或「說個上班的笑話」。

### 對話操作

| 輸入 | 行為 |
| --- | --- |
| `說一個笑話` | 選取素材，生成完整故事 |
| `說個上班的笑話` | 依主題選取素材並生成故事 |
| `換一個`、`好笑一點` | 改選主素材，生成新故事 |
| `笑點在哪？` | 解釋上一個故事的笑點 |
| `然後呢？` | 依上一個故事延伸情節 |
| `clear` | 清空已儲存的對話紀錄 |
| `exit`、`quit`、直接按 Enter | 結束對話 |
| `Ctrl+C` | 中止目前操作並離開 |

### 調整模型與生成設定

模型名稱可透過 `HG_MODEL` 環境變數設定。使用其他模型前，先以 `ollama pull` 下載對應模型。

```powershell
$env:HG_MODEL = "gemma4:e2b"
.\.venv\Scripts\python.exe app.py
```

也可以複製設定範例，再編輯 `.env`：

```powershell
Copy-Item .env.example .env
```

```dotenv
HG_MODEL=gemma4:e2b
```

回答長度可調整 `app.py` 上方的 `MAX_NEW_TOKENS`；生成風格可修改 `SYSTEM_PROMPT` 與故事生成指令。修改後重新啟動程式。

### 連線問題

若無法連線到 Ollama，請先啟動 Ollama 桌面程式，或在另一個終端機執行：

```powershell
ollama serve
```

若顯示找不到模型，先執行 `ollama pull gemma4:e2b`。



[.gitignore](.gitignore) 已排除 `.venv/`、`.runtime/`、本機 `.env` 設定與 Python 快取，並保留 `.env.example`。對話紀錄留在本機，Ollama 模型由各使用者自行下載及管理。
