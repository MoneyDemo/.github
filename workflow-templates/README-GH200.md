# MoneyDemo 組織 Workflow Templates 說明文件

> **本文件供講師課前準備使用。**  
> 語言：繁體中文（台灣），技術術語保留英文原文。

---

## 一、什麼是 Org Workflow Templates？

**Org Workflow Templates**（組織工作流程範本）是一種讓組織管理員（organization admin）預先定義好常用 CI/CD 流程的機制。成員在建立新 workflow 時，可以從組織提供的範本清單中選擇並套用，**不需要從頭撰寫 `.yml` 設定檔**。

每個範本由兩個配對檔案構成：

| 檔案 | 說明 |
|------|------|
| `<name>.yml` | 工作流程本體（標準 GitHub Actions workflow 語法） |
| `<name>.properties.json` | Metadata：範本名稱、說明、圖示、分類（`categories`）及觸發偵測規則（`filePatterns`） |

`properties.json` 必填欄位：`name`、`description`。  
選填欄位：`iconName`（Octicon 或自訂 SVG）、`categories`、`filePatterns`。

---

## 二、存放位置

這些範本**必須**存放在組織的特殊儲存庫 `.github`，且位於頂層的 `workflow-templates/` 目錄：

```
MoneyDemo/
└── .github/                          ← 組織層級的特殊 repository
      └── workflow-templates/         ← 目錄名稱固定，不可更改
            ├── angular-ci.yml
            ├── angular-ci.properties.json
            ├── react-ci.yml
            ├── react-ci.properties.json
            ├── node-ci.yml
            ├── node-ci.properties.json
            ├── python-ci.yml
            ├── python-ci.properties.json
            ├── java-maven-ci.yml
            ├── java-maven-ci.properties.json
            ├── dotnet-ci.yml
            └── dotnet-ci.properties.json
```

> **參考官方文件：**  
> https://docs.github.com/en/actions/how-tos/reuse-automations/create-workflow-templates  
> （本文件已於 2026-09-03 驗證，請上課前確認連結仍有效。）

---

## 三、學員如何套用範本（使用流程）

1. 進入目標 repository，點選上方的 **Actions** tab
2. 點選 **「New workflow」**（右上角或頁面中央）
3. 向下捲動，在 **「MoneyDemo」** 組織區塊找到想使用的範本
4. 點選 **「Configure」**
5. GitHub 會將 `.yml` **複製**到該 repo 的 `.github/workflows/` 目錄，  
   並自動將 `$default-branch` 替換為該 repo 實際的預設分支名稱（例如 `main`）
6. 視需要修改後，commit 即可啟用工作流程

---

## 四、與其他複用機制的差異

這是講師常被學員問到的問題，請務必釐清：

| 機制 | 核心概念 | 複用方式 | 維護模式 |
|------|---------|----------|---------|
| **Workflow Template**（本次） | 範本 → 套用後複製 | 各 repo 持有獨立一份 `.yml` | 各自維護，互不影響 |
| **Reusable Workflow** | 中央工作流程 → 持續呼叫 | `uses: org/.github/.github/workflows/reuse.yml@main` | 集中維護，更新立即生效 |
| **Composite Action** | 封裝多個步驟為單一 `uses:` | `uses: org/my-action@v1` | 集中維護，版本號控制 |

### 關鍵差異（講師要點）

**Workflow Template 是「一次性複製」：**  
套用後，`.yml` 就屬於學員自己的 repository。日後組織管理員若更新原始範本，**已套用的 repository 不會自動跟進**。範本只是一個起點（starting point）。

**Reusable Workflow 是「持續連結」（central link）：**  
呼叫方每次執行都會從來源 repository 讀取最新的 workflow 定義。適合需要組織統一強制執行的政策流程（例如：所有 repo 都必須跑同一份安全掃描）。

> ⚠️ **本次課程（GH-200）的 M2 範圍只教 Workflow Templates，不教 Reusable Workflows。**  
> 此說明僅供講師在學員提問時解釋差異。Reusable Workflows 與 Composite Actions 屬於進階主題，非本次交付範圍。

---

## 五、可見性規則（Visibility）

`.github` repository 的可見性（public / private）決定範本的適用範圍：

| `.github` repo 可見性 | 範本可見給誰 |
|----------------------|------------|
| **Public** | 組織內所有 repository（含 public 與 private） |
| **Private** | 組織的 **private** 及 **internal** repository |

> **說明：** 此可見性行為在本次擷取的官方文件頁面（`create-workflow-templates`）中未明確描述。
> 上述說明為 GitHub 已公開記錄的行為，建議講師上課前至官方文件驗證最新狀態，
> 或至 GitHub 討論區（github.com/orgs/community/discussions）確認。

---

## 六、本次提供的範本索引

| 檔案 | 說明 | `filePatterns` 偵測 |
|------|------|-------------------|
| `angular-ci.yml` | Angular 前端 CI（build + Karma headless test） | `angular.json` |
| `react-ci.yml` | React 前端 CI（build + `CI=true npm test`） | `package.json` |
| `node-ci.yml` | Node.js 後端 CI（lint if present + test） | `package.json` |
| `python-ci.yml` | Python 後端 CI（pytest） | `requirements.txt`、`setup.py`、`pyproject.toml` |
| `java-maven-ci.yml` | Java Maven 後端 CI（mvn verify + upload JAR） | `pom.xml` |
| `dotnet-ci.yml` | .NET C# 後端 CI（restore / build / test） | `*.csproj`、`*.sln` |

---

## 七、Action 版本驗證紀錄

以下版本由指令 `gh api repos/actions/<repo>/releases/latest --jq .tag_name` 驗證（2026-09-03）：

| Action | 已驗證 latest release | 範本使用版本 | 備註 |
|--------|--------------------|------------|------|
| `actions/checkout` | v7.0.1 | `@v7` | 原指定 v5 → 依驗證更新 |
| `actions/setup-node` | v7.0.0 | `@v7` | 原指定 v4 → 依驗證更新 |
| `actions/setup-python` | v7.0.0 | `@v7` | 原指定 v5 → 依驗證更新 |
| `actions/setup-java` | v6.0.0 | `@v6` | 與原指定一致 ✓ |
| `actions/setup-dotnet` | v6.0.0 | `@v6` | 原指定 v4 → 依驗證更新 |
| `actions/upload-artifact` | v7.0.1 | `@v7` | 原指定 v4 → 依驗證更新 |

所有範本均使用 major version tag（`@v7`、`@v6`）而非釘死在特定 patch 版本，以便自動取得 patch/minor 修正。

---

## 八、課程連結（Module 對應）

- **Module 2 — Consume and Troubleshoot Workflows**：使用 `angular-ci`、`react-ci`（Frontend）；`node-ci`、`python-ci`、`java-maven-ci`、`dotnet-ci`（Backend）
- **Module 4 — Manage GitHub Actions in the Enterprise**：示範如何在 `.github` repo 集中管理這些 Enterprise Templates，並說明可見性規則
- **Module 5 — Least Privilege Permissions**：每個範本均宣告 `permissions: contents: read`，作為教學示例
