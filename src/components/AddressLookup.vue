<template>
  <div>
    <label for="address-search">Find an address:</label>
    <input
      id="address-search"
      type="search"
      autocomplete="off"
      list="results-list"
      @input="handleSearch"
      v-model="searchTerm"
    >
    <datalist id="results-list">
      <option v-for="result in results" :key="result.place_id">{{result.display_name}}</option>
    </datalist>
    <div v-if="selectedAddress">
      <p>Address chosen</p>
      <address v-if="selectedAddress">Selected: {{selectedAddress.display_name}}</address>
    </div>
    <div v-if="!selectedAddress">
      <p>status: {{status}}</p>
    </div>
  </div>
</template>

<script>
const MIN_CHARS = 3;

const URL = "https://nominatim.openstreetmap.org/search";
const searchAddresses = searchTerm => {
  const paramString = `format=json&q=${searchTerm}`;
  return fetch(`${URL}?${paramString}`)
    .then(response => response.json())
    .catch(error => {
      console.error("err", error);
    });
};
export default {
  data: () => {
    return { searchTerm: "", results: [], status: "ready" };
  },
  computed: {
    selectedAddress: function() {
      return this.results.find(
        result => result.display_name === this.searchTerm
      );
    }
  },
  methods: {
    handleSearch: function(event) {
      this.status = "received input";
      const searchTerm = String(this.searchTerm).trim();
      const searchTermLongEnough = searchTerm.length >= MIN_CHARS;
      const noAddressSelected = !this.selectedAddress;
      if (searchTermLongEnough && noAddressSelected) {
        this.status = "searching";
        searchAddresses(searchTerm).then(results => {
          this.results = results;
          this.status = `found ${results.length} matches`;
        });
      }
    }
  }
};
</script>

<style scoped>
input {
  width: 100%;
}
</style>
