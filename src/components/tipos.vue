<template>
    <div class="type-filter-container">
      <div 
        v-for="type in types" 
        :key="type" 
        class="type-icon" 
        :class="{ 'selected': selectedTypes.includes(type) }" 
        @click="toggleType(type)"
        :style="{ backgroundColor: getTypeColor(type) }">
        {{ type }}
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'TypeFilter',
    props: {
      types: Array
    },
    data() {
      return {
        selectedTypes: []
      };
    },
    methods: {
      toggleType(type) {
        const index = this.selectedTypes.indexOf(type);
        if (index === -1) {
          this.selectedTypes.push(type);
        } else {
          this.selectedTypes.splice(index, 1);
        }
        this.$emit('update:selected-types', this.selectedTypes);
      },
      getTypeColor(type) {
        const typeColors = {
          fire: '#F08030',
          water: '#6890F0',
          grass: '#78C850',
          electric: '#F8D030',
          ice: '#98D8D8',
          fighting: '#C03028',
          poison: '#A040A0',
          ground: '#E0C068',
          flying: '#A890F0',
          psychic: '#F85888',
          bug: '#A8B820',
          rock: '#B8A038',
          ghost: '#705898',
          dragon: '#7038F8',
          dark: '#705848',
          steel: '#B8B8D0',
          fairy: '#EE99AC'
        };
        return typeColors[type] || '#68A090';
      }
    }
  }
  </script>
  
  <style scoped>
  .type-filter-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    padding: 10px 0;
  }
  
  .type-icon {
    cursor: pointer;
    padding: 5px 10px;
    margin: 5px;
    border-radius: 10px;
    color: white;
    font-weight: bold;
    text-transform: capitalize;
    opacity: 0.5;
    transition: opacity 0.3s ease;
  }
  
  .type-icon.selected {
    opacity: 1;
  }
  </style>
  