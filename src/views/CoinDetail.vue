<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <bounce-loader :loading="isLoading" :color="`#68d391`" :size="150" />
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col lg:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            class="w-20 h-20 mr-5"
            :src="`https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`"
            :alt="asset.name"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500">{{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span class="font-bold">{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Actual Price</b>
              <span class="font-bold">{{ asset.priceUsd | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Lowest Price</b>
              <span class="font-bold">{{ min | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Highest Price</b>
              <span class="font-bold">{{ max | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Average Price</b>
              <span class="font-bold">{{ avg | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">24 hour Variation</b>
              <span
                class="font-bold"
                :class="
                  asset.changePercent24Hr.includes('-')
                    ? 'text-red-500'
                    : 'text-green-400'
                "
                >{{ asset.changePercent24Hr | percent }}</span
              >
            </li>
          </ul>
        </div>
        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            @click="toggleConverter"
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
          >
            {{ fromUsd ? `USD to ${asset.symbol}` : `${asset.symbol} to USD` }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                v-model="convertValue"
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
                :placeholder="
                  fromUsd ? `Amount in USD` : `Amount in ${asset.symbol}`
                "
              />
            </label>
          </div>

          <span class="text-xl">
            {{ convertResult }} {{ fromUsd ? asset.symbol : `USD` }}
          </span>
        </div>
      </div>
      <line-chart
        class="my-5"
        :colors="['orange']"
        :min="min"
        :max="max"
        :data="history.map((h) => [h.date, parseFloat(h.priceUsd).toFixed(2)])"
      />
      <h3 class="text-xl my-10">Best Exchange Offers</h3>
      <table>
        <tr
          v-for="m in markets"
          :key="`${m.exchangeId}-${m.priceUsd}`"
          class="border-b"
        >
          <td>
            <b>{{ m.exchangeId }}</b>
          </td>
          <td>{{ m.priceUsd | dollar }}</td>
          <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
          <td>
            <px-button
              :isLoading="m.isLoading || false"
              v-if="!m.url"
              @click="getWebSite(m)"
            >
              <slot>Get Link</slot>
            </px-button>
            <a v-else class="hover:underline text-green-600" target="_blanck">
              {{ m.url }}
            </a>
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import api from "@/api";
import PxButton from "@/components/PxButton.vue";
export default {
  name: "CoinDetail",
  components: { PxButton },
  data() {
    return {
      asset: {},
      history: [],
      isLoading: false,
      markets: [],
      fromUsd: true,
      convertValue: null,
    };
  },
  created() {
    this.getCoin();
  },
  watch: {
    $route() {
      this.getCoin();
    },
  },
  computed: {
    // eslint-disable-next-line vue/return-in-computed-property
    convertResult() {
      if (!this.convertValue) {
        return 0;
      }
      const result = this.fromUsd
        ? this.convertValue / this.asset.priceUsd
        : this.convertValue * this.asset.priceUsd;
      return result.toFixed(4);
    },
    min() {
      return Math.min(
        ...this.history.map((h) => parseFloat(h.priceUsd).toFixed(2))
      );
    },
    max() {
      return Math.max(
        ...this.history.map((h) => parseFloat(h.priceUsd).toFixed(2))
      );
    },
    avg() {
      return (
        this.history.reduce((a, b) => a + parseFloat(b.priceUsd), 0) /
        this.history.length
      );
    },
  },
  methods: {
    toggleConverter() {
      this.fromUsd = !this.fromUsd;
    },
    getWebSite(exchange) {
      this.$set(exchange, "isLoading", true);
      return api
        .getExchange(exchange.exchangeId)
        .then((res) => {
          this.$set(exchange, "url", res.exchangeUrl);
        })
        .finally(() => {
          this.$set(exchange, "isLoading", false);
        });
    },
    getCoin() {
      this.isLoading = true;
      const id = this.$route.params.id;
      // api.getAsset(id).then((asset) => (this.asset = asset)); original, later changed to host both promises
      Promise.all([
        api.getAsset(id),
        api.getAssetHistory(id),
        api.getMarkets(id),
      ])
        .then(([asset, history, markets]) => {
          (this.asset = asset),
            (this.history = history),
            (this.markets = markets);
        })
        .finally(() => (this.isLoading = false));
    },
  },
};
</script>

<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>
