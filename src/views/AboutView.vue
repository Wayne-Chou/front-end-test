<template>
  <div>
    <form class="search-form">
      <input
        v-model="searchText"
        @input="displayMatches"
        class="search"
        placeholder="City or State"
      />
      <ul class="suggestions" v-html="suggestionsHtml"></ul>
    </form>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      cities: [], //用於存取獲得城市資料
      searchText: "", //用v-model綁定輸入使用者輸入值
      suggestionsHtml: "", //將資料顯示在html上面
    };
  },
  methods: {
    fetchData() {
      const endpoint =
        "https://gist.githubusercontent.com/Miserlou/c5cd8364bf9b2420bb29/raw/2bf258763cdddd704f8ffd3ea9a3e81d25e2c6f6/cities.json";

      // 使用 Axios 發送 GET 請求
      axios
        .get(endpoint)
        .then((response) => {
          // 成功獲取數據後更新本地數據
          this.cities = response.data;
        })
        .catch((error) => {
          console.error("Ajax request failed", error);
        });
    },
    findMatches(wordToMatch, cities) {
      const regex = new RegExp(wordToMatch, "gi");
      return cities.filter(
        (place) => place.city.match(regex) || place.state.match(regex)
      );
    },
    numberWithCommas(x) {
      return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },

    displayMatches() {
      // 當使用者未輸入時不顯示資訊
      if (this.searchText === "") {
        this.suggestionsHtml = "";
        return;
      }

      const matchArray = this.findMatches(this.searchText, this.cities);
      if (matchArray.length === 0) {
        // 如果没有匹配顯示沒有匹配項目
        this.suggestionsHtml = "<li>沒有匹配項目</li>";
        return;
      }
      this.suggestionsHtml = matchArray
        .map((place) => {
          const regex = new RegExp(this.searchText, "gi");
          const cityName = place.city.replace(
            regex,
            `<span class="hl">${this.searchText}</span>`
          );
          const stateName = place.state.replace(
            regex,
            `<span class="hl">${this.searchText}</span>`
          );
          return `
          <li>
            <span class="name">${cityName}, ${stateName}</span>
            <span class="population">${this.numberWithCommas(
              place.population
            )}</span>
          </li>
        `;
        })
        .join("");
    },
  },
  created() {
    // 生命週期create獲取資料
    this.fetchData();
  },
};
</script>

<style>
html {
  box-sizing: border-box;
  background: #ffc600;
  font-family: "helvetica neue";
  font-size: 20px;
  font-weight: 200;
}
*,
*:before,
*:after {
  box-sizing: inherit;
}
input {
  width: 100%;
  padding: 20px;
}

.search-form {
  max-width: 400px;
  margin: 50px auto;
}

input.search {
  margin: 0;
  text-align: center;
  outline: 0;
  border: 10px solid #f7f7f7;
  width: 120%;
  left: -10%;
  position: relative;
  top: 10px;
  z-index: 2;
  border-radius: 5px;
  font-size: 40px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.12), inset 0 0 2px rgba(0, 0, 0, 0.19);
}

.suggestions {
  margin: 0;
  padding: 0;
  position: relative;
  /*perspective:20px;*/
}
.suggestions li {
  background: white;
  list-style: none;
  border-bottom: 1px solid #d8d8d8;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.14);
  margin: 0;
  padding: 20px;
  transition: background 0.2s;
  display: flex;
  justify-content: space-between;
  text-transform: capitalize;
}

.suggestions li:nth-child(even) {
  transform: perspective(100px) rotateX(3deg) translateY(2px) scale(1.001);
  background: linear-gradient(to bottom, #ffffff 0%, #efefef 100%);
}
.suggestions li:nth-child(odd) {
  transform: perspective(100px) rotateX(-3deg) translateY(3px);
  background: linear-gradient(to top, #ffffff 0%, #efefef 100%);
}

span.population {
  font-size: 15px;
}

.hl {
  background: #ffc600;
}

a {
  color: black;
  background: rgba(0, 0, 0, 0.1);
  text-decoration: none;
}
</style>
