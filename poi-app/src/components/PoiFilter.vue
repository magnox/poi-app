<template>
    <div class="poi-filter">
      <h3>Filter POIs</h3>
      <div v-for="type in poiTypes" :key="type" class="filter-option">
        <input type="checkbox" :id="type" :value="type" v-model="selectedTypes">
        <label :for="type">{{ type }}</label>
      </div>
    </div>
  </template>
  
  <script>
export default {
  name: 'PoiFilter',
  props: {
    poiTypes: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      selectedTypes: [],
    };
  },
  created() {
    this.selectedTypes = [...this.poiTypes];
  },
  watch: {
    poiTypes(newVal) {
      this.selectedTypes = [...newVal];
    },
    selectedTypes(newVal) {
      this.$emit('update-filter', newVal);
    },
  },
};
</script>


  
  <style scoped>
  .poi-filter {
    /* Stilisierung der Filterkomponente */
  }
  .filter-option {
    /* Stilisierung der einzelnen Filteroptionen */
  }
  </style>
  