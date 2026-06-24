# FSM 售票機模擬 — Verilog 有限狀態機

> 以有限狀態機(FSM)實作的車站自動售票機模擬,涵蓋投幣、選站、計價、找零與出票流程。數位邏輯 / 系統程式類課堂練習作品。

![Verilog](https://img.shields.io/badge/HDL-Verilog-blue)
![Simulator](https://img.shields.io/badge/sim-Icarus%20Verilog-green)
![License](https://img.shields.io/badge/License-MIT-green)

## 問題(這個程式做什麼)

模擬一台「車站自動售票販賣機」的控制邏輯。使用者投入零錢、選擇起訖站與張數後,機器計算票價、判斷投入金額是否足夠,最後出票並找零。整個流程以一個 **有限狀態機(Finite State Machine)** 驅動,是數位邏輯課程中「用狀態機描述循序控制行為」的典型練習題。

支援的輸入規格:

- **零錢**:1 元、5 元、10 元、50 元(一次只能投入一枚硬幣,不可同時投入多枚)
- **車站**:S1、S2、S3、S4、S5 共五站,需同時選擇 **起站** 與 **終站**;起站與終站相同時即為 **月台票**
- **票數**:1～5 張
- **票價公式**:`票價 = (站距 + 1) × 5`,站距為起訖站編號之差的絕對值

## 方法與技術

本專案核心是一個 **2-bit、四狀態的 FSM**,以 Verilog HDL 描述,模組名稱為 `vending_machine`(檔案 `vending_machine_x.v`)。四個狀態各司其職:

| 狀態 | 名稱 | 行為 |
| --- | --- | --- |
| `S0` | 選站計價 | 依起訖站(`origin` / `destination`)計算單張票價 `costOfTicket = (站距 + 1) × 5`;站號需落在 1～5 才前進 |
| `S1` | 計算總額 | 依張數 `howManyTicket`(1～5)算出應付總額 `moneyToPay = costOfTicket × howManyTicket` |
| `S2` | 投幣累計 | 將投入的 `money` 累加到 `totalMoney`,並提示尚需投入的金額;金額足夠才前進 |
| `S3` | 出票找零 | 印出找零金額與出票張數,並將票價、應付、已投金額歸零,回到 `S0` |

技術重點:

- **狀態暫存器**:`now_state` / `next_state` 兩個 2-bit 暫存器,搭配 `parameter` 定義 `S0`～`S3`。
- **時脈與重置**:以 `posedge clk` 推進狀態;`reset` 將狀態帶回 `S3`(出票/歸零狀態)。
- **資料路徑輸出**:`costOfTicket`(票價)、`moneyToPay`(應付總額)、`totalMoney`(已投金額)三個 7-bit 輸出暫存器。
- **模擬輸出**:以 `$display` 在模擬時印出 `totalMoney`、尚需金額、找零金額與出票張數。

整體屬於 **底層數位邏輯 / HDL** 題材:重點不在演算法,而在用硬體描述語言把一個循序控制流程拆解成狀態、狀態轉移條件,以及每個狀態下的資料運算。

## 結果

完成了一個可在 Verilog 模擬器中運行的售票機 FSM,能依輸入完成「選站 → 算價 → 投幣 → 出票找零」的完整一輪流程,並透過 `$display` 觀察各階段金額變化。

範例(以票價公式推算):

- 起站 `S1`、終站 `S3` → 站距 2 → 單張票價 `(2 + 1) × 5 = 15` 元
- 買 2 張 → 應付總額 `15 × 2 = 30` 元
- 投入足額後 → 出票 2 張,並印出找零金額

> 註:模擬時的逐步輸出由 `$display` 印出(`totalMoney`、`Still need`、`Give changes`、`Tickets`)。

## 如何 build 與執行

本專案以 **Icarus Verilog(iverilog)** 進行編譯與模擬。`vending_machine_x.v` 僅包含設計模組本身,**repo 中尚未附測試平台(testbench)**,需自行撰寫 testbench 提供 `clk`、`reset` 與各輸入訊號來驅動模擬。

安裝 Icarus Verilog 後,典型流程如下:

```bash
# 1. 編譯(把設計檔與你的 testbench 一起編進可執行檔)
iverilog -o vending_machine_sim vending_machine_x.v <your_testbench>.v

# 2. 執行模擬,觀察 $display 輸出
vvp vending_machine_sim
```

> `<your_testbench>.v` 為你需自行建立的測試平台檔(內含 `initial` 區塊產生時脈與輸入)。若 testbench 中加入 `$dumpfile` / `$dumpvars`,可進一步用 GTKWave 觀察波形。

## 已知限制

- 屬於 **較基礎的數位邏輯課堂練習**,著重於 FSM 概念示範,功能與健全性有限。
- **未隨附 testbench**,需自行撰寫測試平台才能跑模擬。
- 投幣一次僅能投入一枚硬幣,不支援同時投入多枚。
- 設計檔的時序寫法偏教學取向(例如組合邏輯區塊中混用了時脈與狀態的敏感列、並使用阻塞式指派),用於學習無妨,但不代表可合成的嚴謹 RTL 風格。
- 車站固定為 5 站、票數固定為 1～5 張,參數未做進一步泛化。
