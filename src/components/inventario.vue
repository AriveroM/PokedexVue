<template>
    <div class="inventory-container">
      <div v-for="(quantity, item) in items" :key="item" class="inventory-card">
        <div class="item-header">
          <span class="item-name">{{ itemNames[item] }}</span>
        </div>
        <img :src="itemIcons[item]" :alt="item" />
        <span class="item-quantity">{{ quantity }}</span>
      </div>
    </div>
  </template>
  
  
  <script>
  import axios from 'axios';
  
  export default {
    name: 'Inventory',
    props: ['items'],
    data() {
      return {
        itemNames: {
          pokeball: 'Pokéball',
          greatball: 'Greatball',
          ultraball: 'Ultraball',
          potion: 'Poción',
          elixir: 'Elixir'
        },
        itemIcons: {
          pokeball: '',
          greatball: '',
          ultraball: '',
          potion: '',
          elixir: ''
        }
      };
    },
    async created() {
      await this.loadItemIcons();
    },
    methods: {
      async loadItemIcons() {
        const itemNames = {
          pokeball: 'poke-ball',
          greatball: 'great-ball',
          ultraball: 'ultra-ball',
          potion: 'potion',
          elixir: 'elixir'
        };
        const itemRequests = Object.keys(itemNames).map(name => axios.get(`https://pokeapi.co/api/v2/item/${itemNames[name]}`));
        try {
          const itemResponses = await Promise.all(itemRequests);
          itemResponses.forEach((response, index) => {
            const key = Object.keys(itemNames)[index];
            this.itemIcons[key] = response.data.sprites.default;
          });
        } catch (error) {
          console.error("Error loading item icons: ", error);
        }
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

.inventory {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.inventory-item {
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

.item-quantity {
  font-size: 18px;
}
</style>

