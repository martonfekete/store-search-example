<template>
  <div class="store-filter__box">
    <div class="store-filter input__wrapper">
      <input
        class="store-filter__input input"
        type="text"
        id="cityFilter"
        placeholder="Start typing to search for stores"
        v-model="filterValue"
        @input="getFilteredValues()"
        @change="getFilteredValues()"
        @focus="getFilteredValues()"
      />
      <span
        class="store-filter__input__clear"
        v-if="showOptions || showSearchError"
        @click="hideSuggestionList()"
      >
        &#x2715;
      </span>
      <ul class="store-filter__options" v-if="showOptions || showSearchError">
        <li
          class="store-filter__option"
          v-for="(option, index) in autocompleteOptions.slice(0, optionsCount)"
          :key="index"
          @click="optionClicked(option)"
          v-html="highlightAutocomplete(option)"
        ></li>
        <li
          class="store-filter__option store-filter__more"
          v-if="autocompleteOptions.length > optionsCount"
        >
          {{ optionsCount }} suggestion{{ optionsCount > 1 ? "s" : "" }} of
          {{ autocompleteOptions.length }}
        </li>
        <li
          class="store-filter__option store-filter__more"
          v-if="showSearchError"
        >
          No results. Try another search phrase.
        </li>
      </ul>
    </div>
    <button
      class="store-search__options__button"
      title="Search options"
      @click="showExtendedOptions = !showExtendedOptions"
    >
      <span
        class="store-search__options__toggle"
        :class="{
          'store-search__options__toggle--extended': showExtendedOptions,
        }"
        >&#x276F;</span
      >
    </button>
    <div class="store-search__options" v-if="showExtendedOptions">
      <div class="store-units input__wrapper">
        <label class="store-units__label input__label" for="units"
          >Number of results</label
        >
        <input
          class="store-units__input input"
          type="number"
          id="units"
          min="1"
          max="15"
          v-model="optionsCount"
          @input="getFilteredValues()"
        />
      </div>
      <div class="input__wrapper input__checkbox">
        <input
          type="checkbox"
          id="showCities"
          value="cities"
          v-model="filterOptions"
        />
        <label for="showCities">Include cities</label>
      </div>
      <div class="input__wrapper input__checkbox">
        <input
          type="checkbox"
          id="showStores"
          v-model="filterOptions"
          value="stores"
        />
        <label for="showStores">Include stores</label>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      options: [],
      autocompleteOptions: [],
      filterValue: "",
      showOptions: false,
      showSearchError: false,
      showExtendedOptions: false,
      optionsCount: 10,
      filterOptions: ["cities", "stores"],
    };
  },
  props: ["parentCities", "parentStores"],
  watch: {
    filterOptions: function () {
      this.buildOptionsList();
    },
  },
  methods: {
    buildOptionsList: function () {
      this.options = [];
      if (this.filterOptions.includes("stores")) {
        this.options = this.options.concat(this.parentStores);
      }
      if (this.filterOptions.includes("cities")) {
        this.options = this.options.concat(this.parentCities);
      }
      this.options.sort();
    },
    getFilteredValues: function () {
      const filteredOptions = getFilteredOptions(
        this.options,
        this.filterValue,
        false
      );
      if (this.filterValue.length >= 3) {
        this.showOptions = filteredOptions.length > 1;
        this.showSearchError = filteredOptions.length === 0;
      } else {
        this.hideSuggestionList();
      }
      this.autocompleteOptions = filteredOptions;
    },
    optionClicked: function (value) {
      this.filterValue = value;
      this.getFilteredValues();
      this.hideSuggestionList();
      this.$emit("option-selected", value);
    },
    highlightAutocomplete: function (option) {
      const highlightAsCity = this.parentCities.includes(option);
      return `<span class="store-filter__option__wrapper store-filter__option--highlight${
        highlightAsCity ? "-city" : ""
      }">${option}</span>`;
    },
    hideSuggestionList: function () {
      this.showOptions = this.showSearchError = false;
    },
  },
  beforeMount() {
    this.buildOptionsList();
    this.getFilteredValues();
  },
};

function getFilteredOptions(options, filter) {
  if (filter.length > 0) {
    const uniFilter = filter.toLowerCase();
    return options.filter((option) => option.toLowerCase().includes(uniFilter));
  }
  return options;
}
</script>

<style lang="scss" scoped>
.store-filter {
  position: relative;
  width: 400px;
  &__box {
    display: inline-block;
  }
  &__input {
    &__clear {
      position: absolute;
      top: 7px;
      right: 7px;
      display: block;
      width: 21px;
      height: 21px;
      cursor: pointer;
      border: thin solid #aaa;
      border-radius: 50%;
      transform: scale(0.8);
      font-size: small;
      &:hover {
        background-color: #aaa;
        color: white;
      }
    }
  }
  &__options {
    width: 100%;
    margin: 0;
    padding: 0.2rem 0 0 0;
    background: #eee;
  }
  &__option {
    display: block;
    background: none;
    margin: 0.2rem 0;
    text-align: left;
    &:hover {
      cursor: pointer;
      background: #aaa;
    }
    &:first-of-type {
      margin-top: 0;
    }
    &--highlight {
      font-weight: bold;
    }
  }
  &__more {
    padding: 0.5rem 1rem 0.25rem;
    font-size: 12px;
    &:hover {
      background: unset;
      cursor: default;
    }
  }
}
.store-units {
  &__label {
    display: inline-block;
    margin-right: 1rem;
  }
  &__input {
    display: inline-block;
    width: 40px;
    padding: 0.25rem;
    border-width: 0 0 thin 0;
    background: transparent;
  }
}
.store-search {
  &__options {
    display: flex;
    justify-content: space-between;
    align-content: center;
    margin-top: 1rem;
    padding: 0.25rem;
    background: #eee;
    font-size: small;
    &__button {
      display: inline-block;
      width: 35px;
      height: 35px;
      background: none;
      border: none;
      cursor: pointer;
      &:focus {
        border: none;
        outline: none;
      }
    }
    &__toggle {
      display: block;
      transform: rotate(90deg) translateY(-1px) scale(1.5);
      &--extended {
        display: block;
        transform: rotate(-90deg) translateY(-1px) scale(1.5);
      }
    }
    .input__checkbox {
      display: flex;
      align-items: center;
      label {
        margin-left: 0.25rem;
      }
    }
  }
}
</style>
