<template>
  <div id="app">
    <div class="currency-converter-container">
      <div class="currency-picker-container">
        <CurrenyPicker
          :base="base"
          :amount="fromAmount"
          :subline="sublineFrom"
          :id="'from'"
          :currencyOptions="currencyOptions"
          :handleInputChange="handleInputChange"
          :currencyValue="fromCurrency"
          :handleSelectChange="handleSelectChange"
        />
        <CurrenyPicker
          :base="base"
          :amount="toAmount"
          :id="'to'"
          :currencyOptions="currencyOptions"
          :handleInputChange="handleInputChange"
          :currencyValue="toCurrency"
          :handleSelectChange="handleSelectChange"
          :subline="sublineTo"
        />
        <SwitchButton :click="switchBase" />
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
      base: "from",
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
      this.conversionRate = res.data.rates["USD"];
      this.currencyOptions = Object.keys(res.data.rates);
      this.toAmount = this.fromAmount * this.conversionRate;
    } catch (error) {
      console.error(error);
    }
  },
  methods: {
    handleInputChange(newValue, id) {
      if (this.base === "from") {
        if (id === "from") {
          this.fromAmount = newValue;
          this.toAmount = newValue * this.conversionRate;
        } else {
          this.toAmount = newValue;
          this.fromAmount = newValue / this.conversionRate;
        }
      } else {
        if (id === "from") {
          this.fromAmount = newValue;
          this.toAmount = newValue / this.conversionRate;
        } else {
          this.toAmount = newValue;
          this.fromAmount = newValue * this.conversionRate;
        }
      }
    },
    async handleSelectChange(event, id) {
      const newCurrency = event.target.value;
      try {
        let res;
        if (id === "from") {
          await this.updateFromCurrency(newCurrency);
        } else {
          await this.updateToCurrency(newCurrency);
        }
      } catch (error) {
        console.error(error);
      }
    },
    async updateFromCurrency(value) {
      let res;
      try {
        if (this.base === "from") {
          res = await axios.get(
            `${BASE_URL}?base=${value}&symbols=${this.toCurrency}`
          );
          this.conversionRate = res.data.rates[this.toCurrency];
          this.toAmount = this.fromAmount * res.data.rates[this.toCurrency];
          this.fromCurrency = value;
        } else {
          res = await axios.get(
            `${BASE_URL}?base=${this.toCurrency}&symbols=${value}`
          );
          this.conversionRate = res.data.rates[value];
          this.fromAmount = this.toAmount * res.data.rates[value];
          this.fromCurrency = value;
        }
      } catch (error) {
        console.error(error);
      }
    },
    async updateToCurrency(value) {
      let res;
      try {
        if (this.base === "from") {
          res = await axios.get(
            `${BASE_URL}?base=${this.fromCurrency}&symbols=${value}`
          );
          this.conversionRate = res.data.rates[value];
          this.toAmount = this.fromAmount * res.data.rates[value];
          this.toCurrency = value;
        } else {
          res = await axios.get(
            `${BASE_URL}?base=${value}&symbols=${this.fromCurrency}`
          );
          this.conversionRate = res.data.rates[this.fromCurrency];
          this.fromAmount = this.toAmount * res.data.rates[this.fromCurrency];
          this.toCurrency = value;
        }
      } catch (error) {
        console.error(error);
      }
    },
    async switchBase() {
      let res;
      let dataInstance = {
        fromAmount: this.fromAmount,
        toAmount: this.toAmount,
        fromCurrency: this.fromCurrency,
        toCurrency: this.toCurrency,
      };
      this.fromAmount = dataInstance.toAmount;
      this.toAmount = dataInstance.fromAmount;
      this.fromCurrency = dataInstance.toCurrency;
      this.toCurrency = dataInstance.fromCurrency;
      if (this.base === "from") {
        res = await axios.get(
          `${BASE_URL}?base=${this.toCurrency}&symbols=${this.fromCurrency}`
        );
        this.base = "to";
        this.toCurrency = res.data.base;
        this.conversionRate = res.data.rates[this.fromCurrency];
      } else {
        res = await axios.get(
          `${BASE_URL}?base=${this.fromCurrency}&symbols=${this.toCurrency}`
        );
        this.base = "from";
        this.fromCurrency = res.data.base;
        this.conversionRate = res.data.rates[this.toCurrency];
      }
    },
  },
  computed: {
    sublineFrom() {
      return `1 ${this.fromCurrency} = ${(1 / this.conversionRate).toFixed(
        5
      )} ${this.toCurrency}`;
    },
    sublineTo() {
      return `1 ${this.toCurrency} = ${(1 * this.conversionRate).toFixed(5)} ${
        this.fromCurrency
      }`;
    },
  },
};
</script>
