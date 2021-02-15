<template>
  <div class="wrapper">
    <h2>Stores</h2>
    <h3>Search for stores</h3>
    <Search
      :parentStores="addressNames"
      :parentCities="cities"
      @option-selected="onOptionSelected"
    />
    <div class="store-list">
      <h3>Browse the full list</h3>
      <button @click.prevent="toggleList('stores')">
        {{ show === "stores" ? "Hide" : "Show" }} stores
      </button>
      <button @click.prevent="toggleList('cities')">
        {{ show === "cities" ? "Hide" : "Show" }} cities
      </button>
      <div>
        <div class="stores" v-if="show === 'stores'">
          <ul class="stores__list">
            <li v-for="store in stores" :key="store.uuid">
              {{ store.addressName }}
            </li>
          </ul>
        </div>
        <div class="cities" v-if="show === 'cities'">
          <button class="btn--secondary" @click="sort(cities, 'citiesAsc')">
            Sort {{ citiesAsc ? "Descending" : "Ascending" }}
          </button>
          <ul class="cities__list">
            <li v-for="city in cities" :key="city">{{ city }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import stores from "../stores";
import Search from "./TheStoreSearch";

const cities = getCitiesSource(stores);

export default {
  data() {
    return {
      show: "",
      stores,
      addressNames: [],
      cities,
      citiesAsc: true,
    };
  },
  methods: {
    toggleList(showKey) {
      this.show = this.show === showKey ? "" : showKey;
    },
    sort(inputArray, sortKey) {
      this[sortKey] = !this[sortKey];
      inputArray.sort();
      if (!this[sortKey]) {
        inputArray.reverse();
      }
    },
    onOptionSelected(option) {
      console.log(option, "selected in autocomplete");
    },
  },
  beforeMount() {
    this.addressNames = stores.map((store) => store.addressName);
  },
  components: {
    Search,
  },
};

function getCitiesSource(source) {
  const storeCities = source.map((store) => store.city);
  // filter duplicate cities
  const uniqueCities = storeCities.filter(function (city, i) {
    return storeCities.indexOf(city) === i;
  });
  // sort asc
  uniqueCities.sort();
  return uniqueCities;
}
</script>

<style lang="scss" scoped>
@import "../assets/main.scss";

.cities,
.stores {
  width: 400px;
  margin: 1rem auto;
}

.cities {
  text-align: right;
}

ul {
  list-style: none;
}

li {
  display: block;
  margin: 0.2rem 0;
  padding: 0.2rem 0.5rem;
  background: #fdc513;
  text-align: left;
}

.store-list {
  margin-top: 2rem;
}
button {
  padding: 0.5rem;
  background: none;
  border: thin solid grey;
  margin: 0 1rem 1rem;
  &:hover {
    cursor: pointer;
    background: #fdc513;
    border-color: #fdc513;
  }
  &.btn {
    &--secondary {
      padding: 0.2rem;
      margin: 0.2rem 0;
      &:hover {
        background: #eee;
        border-color: #eee;
      }
    }
  }
}
</style>
