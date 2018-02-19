<template lang='haml'>
%div.container
  %h3 Currency Converter
  %div
    %div
      %span {{ formattedCurrency1Val }}
    %div
      %span {{ formattedCurrency2Val }}
  %div
    %div.form-group
      %input{type: 'text', 'v-model': 'currency1Val'}
      %select{'v-model': 'currency1', 'name': 'currency1'}
        %option{'v-for': 'currency in currenciesList'} {{ currency }}
    %div.field-list
      %input{type: 'text', 'v-model': 'currency2Val'}
      %select{'v-model': 'currency2', 'name': 'currency2'}
        %option{'v-for': 'currency in currenciesList'} {{ currency }}
</template>
<script>
  import axios from 'axios';
  export default {
    data () {
      return {
        currency1: 'INR',
        currency2: 'USD',
        currency1Val: '',
        currency2Val: '',
        currenciesList: [],
        latestConversions: {}
      }
    },
    mounted: function () {
      this.loadCurrencies();
      this.loadLatestConversions();
    },
    computed: {
      formattedCurrency1Val: function() {
        return this.currency1Val + ' ' + this.currency1 + ' equals'
      },
      formattedCurrency2Val: function() {
        return this.currency2Val + ' ' + this.currency2
      }
    },
    watch: {
      currency2Val: _.debounce(function() {
        this.currency1Val = this.getConvertedValue(this.currency2, this.currency1, this.currency2Val)
      }, 500),
      currency1Val: _.debounce(function() {
        this.currency2Val = this.getConvertedValue(this.currency1, this.currency2, this.currency1Val)
      }, 500),
      currency1: _.debounce(function(newVal, oldVal) {
        if(this.currency1 == this.currency2) {
          this.currency2 = oldVal
        }
        this.currency2Val = this.getConvertedValue(this.currency1, this.currency2, this.currency1Val)
      }, 0),
      currency2: _.debounce(function(newVal, oldVal) {
        if(this.currency2 == this.currency1) {
          this.currency1 = oldVal
        }
        this.currency1Val = this.getConvertedValue(this.currency2, this.currency1, this.currency2Val)
      }, 0)
    },
    methods: {
      loadCurrencies: function() {
                  axios.get('https://openexchangerates.org/api/currencies.json')
                    .then(response => {
                        this.currenciesList = Object.keys(response.data)
                        })
                  .catch(error => {
                      console.log(error);
                      })
                },
      loadLatestConversions: function() {
                             axios.get('https://openexchangerates.org/api/latest.json?app_id=1527fa5e919b4b8096712973f7c52aaf')
                               .then(response => {
                                   this.latestConversions = response.data.rates
                                   })
                             .catch(error => {
                                 console.log(error);
                                 })
                           },
      getConvertedValue: function(from_currency, to_currency, value) {
        let from_currency_rate = this.latestConversions[from_currency]
        let to_currency_rate = this.latestConversions[to_currency]
        let conversionFactor = to_currency_rate/from_currency_rate;
        return (value * conversionFactor);
      }
    }
  }
</script>
<style>
.container {
    margin-top: 50px;
    box-shadow: 0 0 30px black;
    padding: 30px;
}
</style>
