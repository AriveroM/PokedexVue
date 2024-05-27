<template>
    <div class="store-container">
      <div v-for="(item, key) in items" :key="key" class="store-card">
        <img :src="item.icon" :alt="item.name" />
        <div class="item-header">
          <span class="item-name">{{ item.name }}</span>
          <div class="item-controls">
            <span class="control-symbol" @click="decrement(key)">-</span>
            <span class="item-quantity">{{ selectedItems[key] }}</span>
            <span class="control-symbol" @click="increment(key)">+</span>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'Store',
    props: ['items'],
    data() {
      return {
        selectedItems: {
          pokeball: 0,
          greatball: 0,
          ultraball: 0,
          potion: 0,
          elixir: 0
        }
      };
    },
    methods: {
      increment(item) {
        if (this.selectedItems[item] < this.items[item].max) {
          this.selectedItems[item]++;
        }
      },
      decrement(item) {
        if (this.selectedItems[item] > 0) {
          this.selectedItems[item]--;
        }
      },
      resetSelection(item) {
        this.selectedItems[item] = 0;
      }
    }
  }
  </script>
  

  
  
  


<style scoped>


#body {
    display:block;
    place-items: center;
}


#app {
  display: contents;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
}

.store {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.store-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.item-icon {
  width: 40px;
  height: 40px;
  margin-right: 10px;
}

.item-name {
  font-weight: bold;
  margin-right: 10px;
}

.item-controls {
  display: flex;
  align-items: center;
}

.item-controls button {
  width: 30px;
  height: 30px;
  margin: 0 5px;
}

.item-quantity {
  font-size: 18px;
  width: 30px;
  text-align: center;
}

button {
  background-color: #fff;
  border: none;
  padding: 10px 20px;
  margin-top: 10px;
  cursor: pointer;
  border-radius: 5px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #ddd;
}
</style>

  