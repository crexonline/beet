<template>
<div class="balances mt-3">
                <p class="mb-1 font-weight-bold small">Balances</p>
                <table class="table small table-striped table-sm">
                    <tbody>                    
                        <tr v-for="balance in balances" :key="balance.id">
                            <td class="text-left"><span class="small">{{balance.prefix}}</span>{{balance.asset_name}}</td>
                            <td class="text-right">{{balance.balance}}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
    </template>
    
<script>
import { Apis } from "bitsharesjs-ws";
import { setInterval } from "timers";
import store from '../store/index.js';

export default {
  name: "Balances",
  data() {
    return {
      rawbalances: [],
      balances: []
    };
  },
  methods: {
    /*
    refresh: function() {
      let self=this;
      this.$data.refresher=setInterval(function() {
        self.getBalances();
      },3000);
    },
    */
    getBalances: async function() {
      let result = await Apis.instance()
        .db_api()
        .exec("get_full_accounts", [[this.$store.state.BeetStore.wallet.accountID], false])
        .then(res => {
          this.$data.rawbalances = res[0][1].balances;
          let neededassets = [];
          for (var i = 0; i < res[0][1].balances.length; i++) {
            neededassets.push(res[0][1].balances[i].asset_type);
          }
          return Apis.instance()
            .db_api()
            .exec("get_objects", [neededassets]);
        });
      this.$data.balances = [];
      for (var i = 0; i < this.$data.rawbalances.length; i++) {
        if (result[i].issuer == "1.2.0") {
          this.$data.balances[i] = {
            asset_type: this.$data.rawbalances[i].asset_type,
            asset_name: result[i].symbol,
            balance: this.$data.rawbalances[i].balance,
            owner: result[i].issuer,
            prefix: "BIT"
          };
        } else {
          this.$data.balances[i] = {
            asset_type: this.$data.rawbalances[i].asset_type,
            asset_name: result[i].symbol,
            rawbalance: this.$data.rawbalances[i].balance,
            balance: this.formatMoney(
              this.$data.rawbalances[i].balance /
                Math.pow(10, result[i].precision),
              result[i].precision
            ),
            owner: result[i].issuer,
            prefix: ""
          };
        }
      }
      return;
    },
    formatMoney: function(n, decimals, decimal_sep, thousands_sep) {
      var c = isNaN(decimals) ? 2 : Math.abs(decimals),
        d = decimal_sep || ".",
        t = typeof thousands_sep === "undefined" ? "," : thousands_sep,
        sign = n < 0 ? "-" : "",
        i = parseInt((n = Math.abs(n).toFixed(c))) + "",
        j = (j = i.length) > 3 ? j % 3 : 0;
      return (
        sign +
        (j ? i.substr(0, j) + t : "") +
        i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + t) +
        (c
          ? d +
            Math.abs(n - i)
              .toFixed(c)
              .slice(2)
          : "")
      );
    }
  },
  mounted() {}
};
</script>
