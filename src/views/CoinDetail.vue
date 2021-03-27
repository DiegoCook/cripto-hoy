<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <propagate-loader :loading="isLoading" :color="'#47f58a'" :size="15" />
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            class="w-20 h-20 mr-5"
            :src="`https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`"
            :alt="asset.name"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 ml-2 text-gray-300">
              {{ asset.symbol }}</small
            >
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span> #{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio actual</b>
              <span>{{ asset.priceUsd | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Mínimo 24hs</b>
              <span>{{ min | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Máximo 24hs</b>
              <span>{{ max | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Promedio 24hs</b>
              <span>{{ avg | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación</b>
              <span
                :class="
                  asset.changePercent24Hr < 0
                    ? 'text-red-600'
                    : 'text-green-600'
                "
                >{{ asset.changePercent24Hr | percent }}</span
              >
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
          >
            Cambiar
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
              />
            </label>
          </div>

          <span class="text-xl"></span>
        </div>
      </div>
      <line-chart
        class="my-10"
        :colors="['#b00', '#666']"
        :min="min"
        :max="max"
        :data="chartData"
      />
    </template>
  </div>
</template>

<script>
import api from "@/api";

export default {
  name: "CoinDetail",

  data() {
    return {
      isLoading: false,
      asset: {},
      history: [],
    };
  },

  computed: {
    min() {
      const historyPrices24h = this.history.map((h) => h.priceUsd);
      const minimum = Math.min(...historyPrices24h);
      return minimum.toFixed(2);
    },

    max() {
      const historyPrices24h = this.history.map((h) => h.priceUsd);
      const maximum = Math.max(...historyPrices24h);
      return maximum.toFixed(2);
    },

    avg() {
      const historyPrices24h = this.history.map((h) => h.priceUsd);
      const sumPrices = historyPrices24h.reduce(
        (p, el) => p + parseFloat(el),
        0
      );
      const avg = sumPrices / historyPrices24h.length;
      return avg;
    },

    chartData() {
      const data = [];
      this.history.map((h) => {
        data.push([h.date, parseFloat(h.priceUsd).toFixed(2)]);
      });
      return data;
    },
  },

  created() {
    this.getCoin();
  },

  methods: {
    getCoin() {
      const id = this.$route.params.id;
      this.isLoading = true;
      Promise.all([api.getAsset(id), api.getAssetHistory(id)])
        .then(
          ([asset, history]) => ((this.asset = asset), (this.history = history))
        )
        .finally(() => (this.isLoading = false));
    },
  },
};
</script>
