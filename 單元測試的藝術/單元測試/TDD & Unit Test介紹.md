# **TDD & Unit Test介紹**

## 以下我們會分成兩部分來討論：
## * Test Driven Development
  - 流程
  - 核心概念
## * Unit Testing
  - 基本準則
  - 應該具備的特性

> ## What is Test Driven Development ?
> TDD is a software development process that relies on the **repetition** of a very **short** development **cycle**.

## 流程
  1. 先寫測試。
  2. 讓測試失敗！
  3. 寫剛剛好程式讓測試通過。
  4. 重複此循環。

## 核心概念
1. 要先讓**測試失敗**。
2. 寫的程式，需通過**所有測試**。
3. 重構程式碼，再讓它通過測試。

> ## What is Unit Testing ?
> Unit Testing is an automated piece of code that invokes the unit of work being tested, and then checks some assumptions about a single end result of that unit.

## 基本準則
  1. 一個測試案例只測一種方法。
  2. 最小的測試單位。
  3. 不與外部 (包括檔案、資料庫、網路、服務、物件、類別) 直接相依。
  4. 不具備邏輯。
  5. 測試案例之間相依性為零。

## 應該具備的特性：FIRST Principles
  1. Fast：快速。
  2. Independent：獨立。
  3. Repeatable：可重複。
  4. Self-Validating：可反應驗證結果。
  5. Timely：及時。

> ## 說明
> - 開發者不會因執行速度，而不想跑測試。
> - 測試不依賴於其他測試。
> - 測試可在任何環境中重複執行，且執行結果一致。
> - 測試輸出應為布林值，判斷是否成功。
> - 測試最好在寫產品程式碼前寫。
