<template>
  <div id="app">
    <div class="currency-converter-container">
      <div class="currency-picker-container">
        <CurrenyPicker
          :base="fromCurrency"
          :conversionRate="conversionRate"
          :amount="fromAmount"
          :currencyValue2="toCurrency"
          :id="'from'"
          :currencyOptions="currencyOptions"
          :handleInputChange="handleInputChange"
          :currencyValue="fromCurrency"
          :handleSelectChange="handleSelectChange"
        />
        <CurrenyPicker
          :base="fromCurrency"
          :conversionRate="conversionRate"
          :amount="toAmount"
          :id="'to'"
          :currencyOptions="currencyOptions"
          :handleInputChange="handleInputChange"
          :currencyValue="toCurrency"
          :currencyValue2="fromCurrency"
          :handleSelectChange="handleSelectChange"
        />
      </div>
    </div>
  </div>
</template>

<script>
import SwitchButton from "./components/SwitchButton";
import CurrenyPicker from "./components/CurrencyPicker";
import axios from "axios";
import "./assets/sass/index.scss";

const BASE_URL = "https://api.exchangeratesapi.io/latest";

export default {
  name: "App",
  components: {
    CurrenyPicker,
    SwitchButton,
  },
  data: function() {
    return {
      fromAmount: 100,
      toAmount: 0,
      fromCurrency: "EUR",
      toCurrency: "USD",
      currencyOptions: [],
      conversionRate: 0,
    };
  },
  async created() {
    try {
      const res = await axios.get(BASE_URL);
      this.fromCurrency = res.data.base;
      this.base = res.data.base;
      this.conversionRate = res.data.rates["USD"];
      this.currencyOptions = Object.keys(res.data.rates);
      this.toAmount = this.fromAmount * this.conversionRate;
    } catch (error) {
      console.error(error);
    }
  },
  methods: {
    handleInputChange(newValue, id) {
      if (id === "from") {
        this.fromAmount = newValue;
        this.toAmount = newValue * this.conversionRate;
      } else {
        this.toAmount = newValue;
        this.fromAmount = newValue / this.conversionRate;
      }
    },
    async handleSelectChange(event, id) {
      const newCurrency = event.target.value;
      try {
        let res;
        if (id === "from") {
          res = await axios.get(
            `${BASE_URL}?base=${newCurrency}&symbols=${this.toCurrency}`
          );
          this.fromCurrency = res.data.base;
          this.conversionRate = res.data.rates[this.toCurrency];
          this.toAmount = this.fromAmount * res.data.rates[this.toCurrency];
        } else {
          res = await axios.get(
            `${BASE_URL}?base=${this.fromCurrency}&symbols=${newCurrency}`
          );
          this.fromCurrency = res.data.base;
          this.conversionRate = res.data.rates[newCurrency];
          this.toAmount = this.fromAmount * res.data.rates[newCurrency];
          this.toCurrency = newCurrency;
        }
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>
