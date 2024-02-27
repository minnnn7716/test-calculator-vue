# 計算機實作
### [DEMO 頁面](https://minnnn7716.github.io/test-calculator-vue/)
使用 CSS + Vue Composition API 撰寫

## 專案指令
### 安裝相關專案
```sh
npm install
```
### 運行環境

```sh
npm run dev
```
### 建構專案
```sh
npm run build
```

## 專案套件
* Node v20.11.0
* Vue v3.4.20
* Vite v5.1.4
* normalize.css v8.0.1

## 專案功能
### 1. 數字鍵
* 初始值為 0
* 按數字能輸出至畫面 output
* output 長度不得大於 10
### 2. Reset
### 3. Del
### 4. 加減乘除
* 當點擊「運算符號」時，output 需先變回 0，此時點擊其他運算符號都不會有動作，需等到點擊「=」或「Reset」後才能做其他運算。
### 5. Enter（=）