<template>
  <div
    class="currency-picker"
    v-bind:class="{ 'currency-picker-active': active }"
  >
    <div class="head">
      <select
        @change="(e) => handleSelectChange(e, id)"
        v-model="currencyValue"
      >
        <option>{{ currencyValue }}</option>
        <option v-for="(name, index) in filteredOptions" :key="index">
          <div v-if="name !== currencyValue">{{ name }}</div></option
        >
      </select>
    </div>
    <div class="body">
      <div class="input-container">
        <p class="currency-symbol">{{ symbol }}</p>
        <input type="number" v-model="value" v-on:keyup="handleChagne" />
      </div>
      <p class="conversion-hint">
        {{ subline }}
      </p>
    </div>
  </div>
</template>

<script>
import symbols from "../Common-Currency";

export default {
  props: {
    active: Boolean,
    amount: Number,
    handleInputChange: Function,
    handleSelectChange: Function,
    id: String,
    currencyValue: String,
    currencyOptions: Array,
    subline: String,
  },
  data() {
    return {
      value: 0,
    };
  },
  methods: {
    handleChagne(event) {
      this.handleInputChange(parseInt(event.currentTarget.value), this.id);
    },
  },
  mounted() {
    this.value = Math.round(this.amount * 100) / 100;
  },
  watch: {
    amount(newAmount) {
      this.value = Math.round(newAmount * 100) / 100;
    },
  },
  computed: {
    filteredOptions() {
      const result = this.currencyOptions.filter(
        (i) => i !== this.currencyValue
      );
      if (this.currencyValue !== "EUR" && result.indexOf("EUR") === -1)
        result.unshift("EUR");
      return result;
    },
    symbol() {
      return symbols[this.currencyValue].symbol;
    },
  },
};
</script>

<style lang="scss" scoped></style>
