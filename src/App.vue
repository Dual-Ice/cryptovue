<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <!-- <div
      class="
        fixed
        w-100
        h-100
        opacity-80
        bg-purple-800
        inset-0
        z-50
        flex
        items-center
        justify-center
      "
    >
      <svg
        class="animate-spin -ml-1 mr-3 h-12 w-12 text-white"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>
    </div> -->
    <div class="container">
      <add-ticker
        :disabled="tooManyTickersAdded"
        :error="error"
        :coins="coinsList"
        @add-ticker="add"
        @error-clear="errorClear" />
      <template v-if="tickers.length">
        <hr class="w-full border-t border-gray-600 my-4" />
        <div class="my-2">
          Фильтр:
          <input
            v-model="filter"
            class="
              h-10
              border-gray-300
              text-gray-900
              focus:outline-none focus:ring-gray-500 focus:border-gray-500
              sm:text-sm
              rounded-md
            "
          />
        </div>
        <div>
          <button
            v-if="currentPage > 1"
            class="
              my-1
              mx-2
              inline-flex
              items-center
              py-2
              px-4
              border border-transparent
              shadow-sm
              text-sm
              leading-4
              font-medium
              rounded-full
              text-white
              bg-gray-600
              hover:bg-gray-700
              transition-colors
              duration-300
              focus:outline-none
              focus:ring-2
              focus:ring-offset-2
              focus:ring-gray-500
            "
            @click="currentPage = currentPage - 1"
          >
            Назад
          </button>
          <button
            v-if="hasNextPage"
            class="
              my-1
              mx-2
              inline-flex
              items-center
              py-2
              px-4
              border border-transparent
              shadow-sm
              text-sm
              leading-4
              font-medium
              rounded-full
              text-white
              bg-gray-600
              hover:bg-gray-700
              transition-colors
              duration-300
              focus:outline-none
              focus:ring-2
              focus:ring-offset-2
              focus:ring-gray-500
            "
            @click="currentPage = currentPage + 1"
          >
            Вперёд
          </button>
        </div>
        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="(ticker, ndx) in paginatedTickers"
            :key="ndx"
            class="
              bg-white
              overflow-hidden
              shadow
              rounded-lg
              border-purple-800 border-solid
              cursor-pointer
            "
            :class="{
              'border-4': selectedTicker === ticker,
              'bg-red-100': !ticker.availability
            }"
            @click="selectTicker(ticker)"
          >
            <div class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ ticker.name }} - USD
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ formatPrice(ticker.price) }}
              </dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              class="
                flex
                items-center
                justify-center
                font-medium
                w-full
                bg-gray-100
                px-4
                py-4
                sm:px-6
                text-md text-gray-500
                hover:text-gray-600 hover:bg-gray-200 hover:opacity-20
                transition-all
                focus:outline-none
              "
              @click.stop="deleteTicker(ticker)"
            >
              <svg
                class="h-5 w-5"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path></svg
              >Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>
      <ticker-graph
        v-if="selectedTicker"
        :ticker="selectedTicker"
        :bar-width="38"
        :graph-values="tickerGraph"
        @hide-graph="hideGraph"
        />
    </div>
  </div
></template>

<script>
import AddTicker from "./components/AddTicker";
import TickerGraph from "./components/TickerGraph"
import { loadCoins, subscribeToTicker, unSubscribeFromTicker } from "./api";
export default {
  name: "App",

  components: {
    AddTicker,
    TickerGraph
  },

  data() {
    return {
      error: false,
      tickers: [],
      selectedTicker: null,
      tickerGraph: [],
      coinsList: [],
      filter: "",
      currentPage: 1,
      perPage: 6,
      tickersListName: "crypto-list",
      maxTickers: 18
    };
  },

  computed: {
    tooManyTickersAdded() {
      return this.tickers.length >= this.maxTickers
    },

    startPage() {
      return (this.currentPage - 1) * this.perPage;
    },

    endPage() {
      return this.currentPage * this.perPage;
    },

    filtredTickers() {
      return this.tickers.filter((ticker) =>
        ticker.name.includes(this.filter.toUpperCase())
      );
    },

    paginatedTickers() {
      return this.filtredTickers.slice(this.startPage, this.endPage);
    },

    hasNextPage() {
      return this.filtredTickers.length > this.endPage;
    },

    pageStateOptions() {
      return {
        filter: this.filter,
        currentPage: this.currentPage
      };
    }
  },

  watch: {
    filter() {
      this.currentPage = 1;
    },

    pageStateOptions(value) {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}?filter=${value.filter}&currentPage=${value.currentPage}`
      );
    },

    paginatedTickers() {
      if (this.paginatedTickers.length === 0 && this.currentPage > 1) {
        this.currentPage -= 1;
      }
    },

    selectedTicker() {
      this.tickerGraph = [];
    },

    tickers() {
      localStorage.setItem(this.tickersListName, JSON.stringify(this.tickers));
    }
  },

  async created() {
    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );

    const keys = ["filter", "currentPage"];
    keys.forEach((key) => {
      if (windowData[key]) {
        this[key] = windowData[key];
      }
    });

    const tickerData = localStorage.getItem("crypto-list");
    if (tickerData) {
      this.tickers = JSON.parse(tickerData);
      this.tickers.forEach(ticker => {
        subscribeToTicker(ticker.name, (newPrice, availability) => this.updateTicker(ticker.name, newPrice, availability));
      })
    }

    const { Data } = await loadCoins();
    this.coinsList = Object.keys(Data);
  },

  methods: {
    formatPrice (price) {
      if (price === '-') {
        return '-'
      }

      return price > 1 ? price.toFixed(2) : price.toPrecision(2);
    },

    updateTicker(tickerName, price, availability = true) {
      const ticker = this.tickers.find(ticker => ticker.name === tickerName)
      ticker.price = price;
      ticker.availability = availability;
      if (this.selectedTicker?.name === ticker.name) {
        this.tickerGraph = [...this.tickerGraph, ticker.price];
      }
    },

    errorClear() {
      this.error = false;
    },

    add(ticker) {
      this.errorClear;
      const existanceTicker = this.tickers.find(
        (el) => el.name === ticker
      );
      if (existanceTicker) {
        this.error = true;
        return;
      }

      const currentTicker = {
        name: ticker,
        price: "-",
        availability: true
      };

      this.tickers = [...this.tickers, currentTicker];
      subscribeToTicker(currentTicker.name, (newPrice, availability) =>
        this.updateTicker(currentTicker.name, newPrice, availability)
      );

      this.filter = "";
    },

    deleteTicker(tickerToRemove) {
      this.tickers = this.tickers.filter((t) => t !== tickerToRemove);
      if (this.selectedTicker === tickerToRemove) {
        this.selectedTicker = null;
      }

      unSubscribeFromTicker(tickerToRemove.name);
    },

    hideGraph() {
      this.selectedTicker = null;
    },

    selectTicker(ticker) {
      this.selectedTicker = ticker;
    }
  }
};
</script>
