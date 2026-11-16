# AI 股市走勢預測_HKIEI

用 Yahoo Finance 已收市日線，在本機模擬港股、美股、指數、ETF 同加密貨幣嘅未來走勢。結果包括整段方向、每日悲觀／中間／樂觀收市情境、SVG 走勢圖同互動 HTML 報告。亦可以用歷史行情回測，了解模型喺指定標的嘅表現。

## 下載

打開 [AI_Stock_Forecast_HKIEI.zip](./AI_Stock_Forecast_HKIEI.zip)，喺 GitHub 檔案頁撳下載圖示，然後解壓。搵到同時有 `plugin.json` 同 `skills` 嘅資料夾。以下步驟用到入面嘅 `skills/hkiei-market-forecast`。

## 首次設定

需要 **Apple Silicon Mac（M1 或更新）**、**Python 3.9 或以上**、網絡連線同可用磁碟空間。首次設定會建立獨立 Python 環境，下載 PyTorch、模型同分詞器。

1. 用 Finder 打開解壓後嘅 `skills`，搵到入面嘅 `hkiei-market-forecast` 資料夾。
2. 開啟「終端機」，輸入 `cd `（`cd` 後面留一格空白），將 Finder 入面嘅 `hkiei-market-forecast` 資料夾拖入終端機，再按 Enter。
3. 執行以下兩行指令。第二行顯示「模型運行環境已就緒」就可以開始使用。

```sh
python3 setup_market_forecast.py
python3 setup_market_forecast.py --check
```

如曾經完成設定，可以先執行第二行檢查。預設運行環境位於 `~/.cache/hkiei-market-forecast/`。

## 喺 ChatGPT、Codex 或 Claude Code 使用

| 平台 | 點樣啟用 |
| --- | --- |
| ChatGPT 桌面版 | 如果你嘅插件列表有 **AI 股市走勢預測_HKIEI**，安裝後喺支援本機執行嘅新對話選取插件。 |
| Codex 本機 | 將整個 `hkiei-market-forecast` 資料夾複製到 `~/.agents/skills/`，喺本機 Codex 選取相應技能。 |
| Claude Code 本機 | 將整個 `hkiei-market-forecast` 資料夾複製到 `~/.claude/skills/`，喺本機 Claude Code 輸入 `/hkiei-market-forecast`。 |

[Codex 技能目錄](https://learn.chatgpt.com/docs/build-skills) · [Claude Code 技能目錄](https://code.claude.com/docs/en/skills)

**可以直接複製呢段提問**

> 用最新已收市日線，預測恒指（^HSI）未來 5 個香港交易日。先講整體方向同第 5 日較最新收市相差幾多點、幾多個百分比。再按日列出預估方向同三個收市情境。標明行情截止日期，核對香港交易日同休市日，附互動 HTML 報告及 SVG 圖。

亦可以輸入其他 Yahoo Finance 代號，例如騰訊 `0700.HK`、NVIDIA `NVDA`、標普 500 `^GSPC`、納指 `^IXIC`、道指 `^DJI` 同比特幣 `BTC-USD`。

## 用終端機輸出 HTML 報告同 SVG

先進入 `skills/hkiei-market-forecast`。以下例子用恒指做 5 日模擬，將結果存到桌面一個新資料夾，完成後會打開 HTML。

```sh
report_dir="$(mktemp -d "$HOME/Desktop/HKIEI_Forecast_XXXXXX")"
python3 market_forecast.py '^HSI' --days 5 --json "$report_dir/forecast.json" --save-raw "$report_dir/history.json" --svg "$report_dir/chart.svg" &&
python3 market_report.py --forecast "$report_dir/forecast.json" --raw "$report_dir/history.json" --output "$report_dir/report.html" &&
open "$report_dir/report.html"
```

| 檔案 | 內容 |
| --- | --- |
| `report.html` | 互動走勢報告，可用瀏覽器打開 |
| `chart.svg` | 獨立走勢圖 |
| `forecast.json` | 預測、模型設定同模擬結果 |
| `history.json` | 同一次運算用到嘅 Yahoo Finance 原始行情，可供離線重播 |

預設參考最近 120 個交易日，模擬 50 次，預估未來 10 個交易日。可用 `--days` 指定預測長度，支援 1 至 30 日。需要直度 SVG 時，加 `--size 1080x1920`。

第 1 日方向以最新實際收市為基準，之後逐日同上一日嘅中間情境比較。整段方向同最後一日變幅都以最新實際收市為基準。

## 做一次回測

以下例子用最近 5 個歷史起點逐日做恒指 1 日回測。每個起點只用當日或之前嘅資料模擬，再同之後嘅實際收市比較。回測結果會顯示喺終端機，同時另存 JSON 到桌面。

```sh
backtest_file="$HOME/Desktop/HKIEI_HSI_Backtest_$(date +%Y%m%d_%H%M%S).json"
python3 market_forecast.py '^HSI' --backtest --days 1 --step 1 --origins 5 --json "$backtest_file"
```

可以核對三項結果：

- 實際收市有幾多次落入悲觀至樂觀情境範圍
- 預估升跌方向有幾多次吻合
- 中間情境嘅收市價誤差，同「收市價維持不變」嘅基準相比有幾大

想將同一標的嘅回測摘要放入 HTML，可喺重新產生報告時向 `market_report.py` 加 `--backtest`，指定回測 JSON，並使用新嘅 HTML 檔名。

## 使用時留意

- 腳本會剔除未收市日線。股票、ETF 同指數嘅未來日期由腳本略過週末推算，公眾假期及休市日要按交易所日曆核對。加密貨幣按連續日曆日計算。
- 模型按歷史行情推算情境。業績、議息及突發消息要另外核對。實際價格可以超出模擬範圍。
- 回測分開報告方向命中、情境覆蓋同收市價誤差。樣本數同所選標的會影響結果。
- 如見到「模型運行環境尚未安裝或不完整」，返到技能資料夾執行 `python3 setup_market_forecast.py`，完成後再用 `--check` 檢查。

模擬結果只供教育及資訊參考，並不構成投資建議。投資涉及風險，歷史表現不代表未來。

## 移除

喺技能資料夾執行 `python3 setup_market_forecast.py --remove`，刪除本機運行環境，再喺所用平台移除插件或技能。

## 第三方元件及授權

模型程式碼、來源記錄及 MIT 授權載於 `skills/hkiei-market-forecast/vendor/`。模型權重及分詞器於安裝時下載。隨附的投創教育標誌只供此插件的報告使用。各套件依原授權使用。

## 版權

版權所有 © 2026 投創教育。保留所有權利（上述第三方元件除外）。

未經投創教育書面同意，不得修改、複製、轉售或再分發本 skill 或其任何部分（上述第三方元件按其原有授權處理）。
