<template>
  <div class="poi-filter">
    <div class="chip" v-for="type in poiTypes" :key="type" :class="{ active: selectedTypes.includes(type) }"
      @click="toggleSelection(type)">
      {{ type }}
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
  methods: {
    toggleSelection(type) {
      const index = this.selectedTypes.indexOf(type);
      if (index > -1) {
        this.selectedTypes.splice(index, 1); // Entferne Typ, wenn bereits ausgewählt
      } else {
        this.selectedTypes.push(type); // Füge Typ hinzu, wenn nicht ausgewählt
      }
      this.$emit('update-filter', this.selectedTypes);
    },
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
.chip {
  display: inline-block;
  padding: 5px 10px;
  border: 1px solid #ccc;
  border-radius: 25px;
  margin: 2px;
  cursor: pointer;
  transition: background-color 0.3s, color 0.3s;
  font-family: Sans-Serif;
}

.chip.active {
  background-color: #007BFF;
  color: white;
  border-color: #007BFF;
}
</style>
  