# AI 股市走勢預測_HKIEI

輸入股票或指數的名稱及預測日數，插件便會根據 Yahoo Finance 提供的已收市日線數據，估算未來每日走勢，並以表格和圖表顯示結果。支援港股、美股、指數、ETF 及加密貨幣。

## 使用示例

已在 ChatGPT 桌面版安裝插件的使用者，可在新對話選取 **AI 股市走勢預測_HKIEI**，然後直接複製以下提問。首次使用請參閱下方的安裝步驟。

> 請預測恒指（^HSI）未來 5 個香港交易日的走勢。每個交易日都要有三個收市估算，分別是偏低、中間和偏高。
>
> - 先說明這 5 日整體偏升還是偏跌。將第 5 日的中間收市估算與最新實際收市比較，列出預計升跌多少點及升跌幅（%）。
> - 用表格逐日列出日期、預期升跌方向，以及該日的三個收市估算。
> - 列明數據截至哪一天，按港股交易日曆核對日期，並附上互動 HTML 報告和 SVG 走勢圖。

模型會模擬多種走勢，再為每個交易日整理出偏低、中間和偏高的收市估算。預測 5 日，表格便有 5 行，每行 3 個收市數字。

舉例來說，假設恒指最新實際收報 25,000 點，第 5 日的三個估算為 24,800、25,200 和 25,600 點。計算整段預測的升跌幅時，採用中間的 25,200 點，即預計到第 5 日上升 200 點（+0.8%）。

預測其他股票、指數或資產時，只需更改名稱或代號，以及預測日數。常用代號包括騰訊 `0700.HK`、NVIDIA `NVDA`、標普 500 指數 `^GSPC`、納指 `^IXIC`、道指 `^DJI` 及比特幣 `BTC-USD`。

## 首次安裝

請準備一部 **配備 Apple 晶片的 Mac（M1 或更新型號）**、**Python 3.9 或以上版本**、網絡連線及可用磁碟空間。

1. 開啟 [AI_Stock_Forecast_HKIEI.zip](./AI_Stock_Forecast_HKIEI.zip)，按 GitHub 檔案頁右上方的下載圖示。下載後，雙擊 ZIP 檔案解壓。
2. 在 Finder 開啟解壓後的 `skills` 資料夾，找到 `hkiei-market-forecast`。開啟 Mac 的「終端機」，輸入 `cd` 後按一下空白鍵，將 `hkiei-market-forecast` 資料夾拖到終端機視窗，再按 Enter。
3. 複製以下兩行指令到終端機執行。首次安裝會下載模型及所需檔案。終端機顯示「模型運行環境已就緒」，即表示安裝完成。日後只需執行第二行，便可檢查安裝狀態。

```sh
python3 setup_market_forecast.py
python3 setup_market_forecast.py --check
```

### 安裝後如何使用

- 如 **ChatGPT 桌面版**的插件列表有 **AI 股市走勢預測_HKIEI**，請安裝插件，再在支援本機執行的新對話選取它。
- 使用 **Codex 本機版**時，將整個 `hkiei-market-forecast` 資料夾放入 `~/.agents/skills/`，再在 Codex 選取這項技能。
- 使用 **Claude Code 本機版**時，將整個 `hkiei-market-forecast` 資料夾放入 `~/.claude/skills/`，再在 Claude Code 輸入 `/hkiei-market-forecast`。

也可直接在 Mac 終端機執行下方的指令。[Codex 技能目錄](https://learn.chatgpt.com/docs/build-skills) · [Claude Code 技能目錄](https://code.claude.com/docs/en/skills)

## 如何閱讀預測結果

- **三個收市估算**分別是偏低、中間和偏高，對應報告中的「悲觀」、「中間」和「樂觀」情境。
- **每日升跌方向**按中間估算判斷。第 1 日與最新實際收市比較，第 2 日起與前一日的中間估算比較。整體走勢及最後一日的變幅均以最新實際收市為基準。
- **交易日期**須按交易所日曆核對。終端機指令會略過週末，公眾假期及其他休市日須另行核對。加密貨幣的預測日期按日曆日連續排列。

## 如何查看以往的預測表現

回測是用當時已知的數據作出預測，再將結果與之後的實際收市比較。可參考以下提問。

> 請用最近 5 個交易日做恒指單日回測。列出有多少次估中升跌方向、多少次實際收市落在預測範圍內，以及預測收市點數與實際收市相差多少。再與「收市點數維持不變」的估算比較，看看哪種方法較接近實際結果。

## 終端機指令

<details>
<summary>恒指 5 日預測，輸出 HTML 報告及 SVG 圖</summary>

先在終端機進入 `skills/hkiei-market-forecast` 資料夾，再執行以下指令。HTML 報告、SVG 走勢圖、預測數據及 Yahoo Finance 原始數據會存入桌面上的新資料夾。完成後，瀏覽器會開啟 HTML 報告。

```sh
report_dir="$(mktemp -d "$HOME/Desktop/HKIEI_Forecast_XXXXXX")"
python3 market_forecast.py '^HSI' --days 5 --json "$report_dir/forecast.json" --save-raw "$report_dir/history.json" --svg "$report_dir/chart.svg" &&
python3 market_report.py --forecast "$report_dir/forecast.json" --raw "$report_dir/history.json" --output "$report_dir/report.html" &&
open "$report_dir/report.html"
```

可用 `--days` 設定預測日數，範圍為 1 至 30 日。按預設設定，程式會參考最近 120 個交易日的數據，進行 50 次模擬，預測未來 10 個交易日。如需直度 SVG 圖，可在 `market_forecast.py` 的指令加入 `--size 1080x1920`。

</details>

<details>
<summary>恒指單日回測，查看最近 5 次結果</summary>

在同一個技能資料夾執行以下指令。結果會在終端機顯示，並以 JSON 檔案另存於桌面。

```sh
backtest_file="$HOME/Desktop/HKIEI_HSI_Backtest_$(date +%Y%m%d_%H%M%S).json"
python3 market_forecast.py '^HSI' --backtest --days 1 --step 1 --origins 5 --json "$backtest_file"
```

如需將回測摘要加入同一股票或指數的 HTML 報告，可在 `market_report.py` 的指令加入 `--backtest "$backtest_file"`，並使用新的 HTML 檔名。

</details>

<details>
<summary>安裝問題及移除方法</summary>

如果終端機顯示「模型運行環境尚未安裝或不完整」，請在技能資料夾重新執行 `python3 setup_market_forecast.py`，完成後以 `--check` 檢查。程式預設將運行環境存於 `~/.cache/hkiei-market-forecast/`。

如需移除，請在技能資料夾執行 `python3 setup_market_forecast.py --remove`，然後在所用程式移除插件或技能。

</details>

## 第三方元件及授權

模型程式碼、來源記錄及 MIT 授權載於 `skills/hkiei-market-forecast/vendor/`。模型權重及分詞器於安裝時下載。隨附的投創教育標誌只供此插件的報告使用。各套件依原授權使用。

## 版權及使用權限

版權所有 © 2026 投創教育（第三方元件除外）。

投創教育允許使用者從本 GitHub 專案下載插件，並在自己的電腦安裝、執行及保存副本，供個人非商業用途。此許可適用於本專案提供的 ZIP 檔案。下載包內如有舊版版權聲明，下載及使用權限以本節為準。

歡迎分享本專案或官方下載頁的連結。修改、轉售、另行分發插件檔案或作商業用途，須事先取得投創教育的書面同意。投創教育保留本節未授予的權利。

在 GitHub 平台瀏覽及 Fork 的權利依 [GitHub 服務條款](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service#5-license-grant-to-other-users)。第三方元件依各自原有授權使用。

## 免責聲明

本插件的走勢預測及模擬情境由 **AI 模型根據過往日線數據生成**。預測結果、圖表及報告僅供教育及一般資訊參考。

本插件提供的內容不構成投資建議或買賣推薦，亦不構成買賣、認購或投資於任何證券、基金、虛擬資產或其他投資產品的要約、招攬或邀請。

投資涉及風險。AI 預測可能有誤，市場數據亦可能有延遲。歷史表現及回測結果不代表未來表現。使用者應自行核對資料並作出獨立判斷。
