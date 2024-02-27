<script setup>
import { ref, watchEffect } from "vue";

const initData = {
  numBtn: sortNumbers(),
  fnBtn: [
    {
      name: "Reset",
      class: "flex-flexible btn-fn",
    },
    {
      name: "Del",
      class: "btn-sm btn-fn",
    },
  ],
  calcBtn: ["/", "*", "-", "+", "="],
};

const calcFn = {
  "+": (num1, num2) => num1 + num2,
  "-": (num1, num2) => num1 - num2,
  "*": (num1, num2) => num1 * num2,
  "/": (num1, num2) => num1 / num2,
  "=": (ary) => {
    let total = 0;

    for (let i = 0; i < ary.length; i += 1) {
      if (i === 0) {
        const firstNum = ary[i];
        total = firstNum;
      }

      if (typeof ary[i] === "string") {
        const calcType = ary[i];
        const nextNum = ary[i + 1];
        total = calcFn[calcType](total, nextNum);
      }
    }

    if (`${total}`.includes(".") && `${total}`.split(".")[1].length > 10) {
      total = Number(total.toFixed(10));
    }

    return total;
  },
};

const currentNum = ref(0);
const currentCalc = ref("");
const calcProcess = ref([]);
const disableFn = ref(false);
const disableResult = ref(false);
const smaller = ref(false);
const alertText = ref(false);

// 排序數字鍵盤渲染資料
function sortNumbers() {
  let wholeNumbers = [];
  let rowNumbers = [];

  for (let i = 0; i <= 9; i += 1) {
    if (i % 3 === 0) {
      rowNumbers.push(i);
      wholeNumbers = [...rowNumbers, ...wholeNumbers];
      rowNumbers = [];
    } else {
      rowNumbers.push(i);
    }
  }

  return wholeNumbers;
}

function clickFn(action) {
  const { value } = currentNum;
  let num = value.toString();

  if (action === "Del" && currentCalc.value === "=") {
    return;
  }

  if (action === "Del" && num.length > 1) {
    currentNum.value = Number(num.slice(0, num.length - 1));
  } else {
    currentNum.value = 0;
  }

  if (action === "Reset") {
    calcProcess.value = [];
    currentNum.value = 0;
  }

  currentCalc.value = action;
  setTimeout(() => {
    currentCalc.value = "";
  }, 1000);
}

function clickNum(action) {
  const { value } = currentNum;
  let num = "";

  if (value && currentCalc.value !== "=") {
    num = value.toString();
  }

  if (currentCalc.value === "=") {
    currentCalc.value = "";
  }

  if (num.length < 10) {
    num += action;
    currentNum.value = Number(num);
  } else {
    alertText.value = true;

    setTimeout(() => {
      alertText.value = false;
    }, 2000);
  }
}

function clickCalc(action) {
  const { value } = currentNum;

  if (action === "=") {
    calcProcess.value.push(value);
    currentCalc.value = action;

    let total = calcFn["="](calcProcess.value);

    if (!isFinite(total) && !isNaN(total)) {
      total = "無法除以零";
    } else if (isNaN(total)) {
      total = "無法計算";
    }

    currentNum.value = total;
    calcProcess.value = [];
    return;
  }

  calcProcess.value.push(value, action);
  currentNum.value = 0;
  currentCalc.value = action;
}

// 依據情況 disable 運算按鈕（不可重複點擊運算符號、運算結果不為數字）
watchEffect(() => {
  const lastIndex = calcProcess.value.length - 1;

  if (typeof calcProcess.value[lastIndex] === "string" && !currentNum.value) {
    disableFn.value = true;
    return;
  }

  if (typeof currentNum.value !== "number") {
    disableFn.value = true;
    disableResult.value = true;
    return;
  }

  disableFn.value = false;
  disableResult.value = false;
});

// 當運算結果大於 12 字元，縮小字級
watchEffect(() => {
  if (`${currentNum.value}`.length >= 12) {
    smaller.value = true;
  } else {
    smaller.value = false;
  }
});
</script>

<template>
  <div class="wrap">
    <div class="calculator">
      <p class="alert-text" :class="{ show: alertText }">最多僅能輸入 10 個數字</p>
      <div class="calculator-output d-flex justify-between">
        <span class="calculator-output-calc">{{ currentCalc }}</span>
        <div class="flex-flexible text-end">
          <p class="calculator-output-process">{{ calcProcess.join("") }}</p>
          <h1
            class="calculator-output-num d-flex align-items-center justify-end"
            :class="{ smaller: smaller }"
          >
            {{ currentNum }}
          </h1>
        </div>
      </div>
      <div class="calculator-buttons d-flex justify-between gap-10">
        <div class="calculator-main d-flex flex-column flex-flexible gap-10">
          <div class="calculator-fnButtons d-flex justify-between gap-10">
            <button
              v-for="item in initData.fnBtn"
              type="button"
              :key="`fnBtn ${item.name}`"
              :class="item.class"
              @click="clickFn(item.name)"
            >
              {{ item.name }}
            </button>
          </div>
          <div class="calculator-numButtons d-flex flex-wrap justify-between gap-10">
            <button
              v-for="item in initData.numBtn"
              type="button"
              :key="`numBtn ${item}`"
              :class="{
                'btn-sm': item !== 0,
                'btn-lg': item === 0,
              }"
              @click="clickNum(item)"
            >
              {{ item }}
            </button>
          </div>
        </div>
        <div
          class="calculator-calcButtons d-flex flex-column gap-10"
          :class="{
            disable: disableFn,
            disableResult: disableResult,
          }"
        >
          <button
            v-for="item in initData.calcBtn"
            type="button"
            class="btn-lg"
            :key="`calcBtn ${item}`"
            :class="{
              'btn-fn': item !== '=',
              'btn-result': item === '=',
            }"
            @click="clickCalc(item)"
          >
            {{ item }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.wrap {
  padding: 0 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  font-size: 1.5em;
  font-family: "Noto Sans TC", sans-serif;

  @media screen and (max-width: 320px) {
    font-size: 1.2em;
  }
}

.calculator {
  position: relative;
  margin: 48px 0;
  padding: 30px;
  max-width: 500px;
  border: 5px solid #425069;
  border-radius: 10px;
  background-color: #697ca0;

  @media screen and (max-width: 550px) {
    padding: 20px;
  }
}

.calculator-output {
  padding: 0 16px;
  margin-bottom: 10px;
  border: 2px solid #000;
  border-radius: 5px;
  background-color: #fff;

  @media screen and (max-width: 550px) {
    padding: 0 8px;
  }
}

.calculator-output-num {
  margin: 4px 0 24px 0;
  letter-spacing: 1px;
  min-height: 55px;

  @media screen and (max-width: 550px) {
    min-height: 40px;
    font-size: 1.5em;
  }
}

.calculator-output-num.smaller {
  font-size: 1.2em;

  @media screen and (max-width: 550px) {
    font-size: 0.8em;
  }
}

.calculator-output-process {
  height: 20px;
  margin: 8px 0 0 0;
  font-size: 0.8em;
  letter-spacing: 1px;
  color: #9c9c9c;
}

.calculator-output-calc {
  margin: 8px 0;
  font-size: 1em;
  color: #e69468;
}

.calculator-calcButtons {
  width: 25%;
  max-width: 100px;
}

.disable .btn-fn {
  color: #5c5c5c;
  border-color: #585858;
  background-color: #9c9c9c;
  box-shadow: none;
  pointer-events: none;
}

.disable .btn-fn:hover {
  margin-top: 0;
  margin-bottom: 0;
  background-color: #9c9c9c;
}

.disableResult .btn-result {
  color: #5c5c5c;
  border-color: #585858;
  background-color: #9c9c9c;
  box-shadow: none;
  pointer-events: none;
}

.disableResult .btn-result:hover {
  margin-top: 0;
  margin-bottom: 0;
  background-color: #9c9c9c;
}

button {
  height: 60px;
  border: 2px solid #000;
  border-radius: 5px;
  box-shadow: 0 3px 0px #000;
  cursor: pointer;
  transition: all ease 0.5s;
}

button:hover {
  margin-top: 3px;
  margin-bottom: -3px;
  background-color: #d4dbe7;
  box-shadow: 0 0 0px #000;
}

.btn-result {
  background-color: #e69468;
}

.btn-result:hover {
  background-color: #d37c4d;
}

.btn-fn {
  background-color: #ece2c7;
}

.btn-fn:hover {
  background-color: #ebd69d;
}

.alert-text {
  min-width: 250px;
  position: absolute;
  top: -45px;
  left: 50%;
  transform: translateX(-50%);
  margin: 0;
  padding: 8px 16px;
  border-radius: 40px;
  text-align: center;
  font-size: 16px;
  color: #631a26;
  background-color: #ffd1d9;
  opacity: 0;
  transition: opacity ease 0.3s;
}

.alert-text.show {
  opacity: 100%;
}
</style>
