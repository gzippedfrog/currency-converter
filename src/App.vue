<script setup>
import { ref, onBeforeMount, reactive, watch } from "vue";
import localeCurrency from "locale-currency";

const API_URL = "https://api.apilayer.com/fixer";
const API_KEY = "6D3VTqeMGcnl5MnfKojj3SuFBLOmAjul";

const language = ref(window.navigator.language || "en-US");
const base = ref();
const target = ref("RUB");
const amount = ref(1);

const currencies = reactive({
  loading: false,
  list: {}
});

const result = reactive({
  loading: false,
  value: "",
});

function wheelScrollHandler(e) {
  if (e.wheelDeltaY > 0) {
    amount.value++;
  } else {
    amount.value--;
  }
}

function fetchCurrencies() {
  currencies.loading = true;
  return fetch(`${API_URL}/symbols`, {
    headers: {
      apikey: API_KEY
    }
  })
    .then((res) => res.json())
    .then((data) => {
      currencies.list = data.symbols;
      base.value = localeCurrency.getCurrency(language.value.split("-")[1]);
    })
    .catch((err) => {
      console.log(err.message);
    })
    .finally(() => {
      currencies.loading = false;
    });
}

function fetchExchangeRate() {
  if (!base.value || !target.value || !amount.value) return;
  result.loading = true;
  fetch(
    `${API_URL}/convert?` +
      new URLSearchParams({
        from: base.value,
        to: target.value,
        amount: amount.value
      }),
    {
      headers: {
        apikey: API_KEY
      }
    }
  )
    .then((res) => res.json())
    .then((data) => {
      result.value = data.result;
    })
    .catch((err) => console.log(err.message))
    .finally(() => {
      result.loading = false;
    });
}

onBeforeMount(() => {
  fetchCurrencies().then(fetchExchangeRate);
});

watch([base, target, amount], fetchExchangeRate);
</script>

<template>
  <h1>Currency converter</h1>

  <label for="baseCurrency">From</label>
  <select
    v-if="currencies.loading"
    name="baseCurrency"
    id="baseCurrency"
    disabled
  >
    <option value="" selected>Loading...</option>
  </select>

  <select v-else name="baseCurrency" id="baseCurrency" v-model="base">
    <option v-for="(item, key) in currencies.list" :value="key">
      {{ item }}
    </option>
  </select>

  <label for="targetCurrency">To</label>
  <select
    v-if="currencies.loading"
    name="targetCurrency"
    id="targetCurrency"
    disabled
  >
    <option value="" selected>Loading...</option>
  </select>

  <select v-else name="targetCurrency" id="targetCurrency" v-model="target">
    <option v-for="(item, key) in currencies.list" :value="key">
      {{ item }}
    </option>
  </select>

  <label for="amount">Amount</label>
  <input
    type="number"
    name="amount"
    id="amount"
    v-model="amount"
    @wheel="wheelScrollHandler"
    :disabled="result.loading"
  />

  <hr />
  <div>Result</div>
  <div class="result">{{ result.loading ? "Loading..." : result.value }}</div>
</template>

<style>
html,
body {
  height: 100%;
  margin: 0;
}

body {
  background: linear-gradient(#b868c2, #4080a5);
  color: rgba(255, 255, 255, 0.8);
  font-weight: bold;
}

#app {
  display: grid;
  align-items: center;
  align-content: center;
  grid-template-columns: 1fr 1fr;
  row-gap: 10px;
  font-family: sans-serif;
  max-width: 600px;
  margin: 0 auto;
  border: solid transparent 1px;
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0 0 10px black;
  background-color: rgba(0, 0, 0, 0.5);
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

h1 {
  grid-column: 1/3;
  text-align: center;
  margin-top: 0;
}

input,
select {
  border: solid transparent 10px;
  border-radius: 3px;
  background-color: rgba(255, 255, 255, 0.3);
  color: white;
}

option {
  color: black;
}

hr {
  grid-column: 1/3;
  width: 100%;
  border: none;
  border-top: solid white 1px;
}
</style>
