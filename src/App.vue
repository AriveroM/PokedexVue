<template>
  <div>
    <header class="app-header">
      <h1>Mi Pokédex</h1>
      <nav>
        <button @click="goToPokedex">Pokédex</button>
        <button @click="goToInventory">Inventario</button>
        <button @click="goToStore">Tienda</button>
        <button v-if="view === 'pokedex'" @click="toggleTeamFilter">{{ showTeamOnly ? 'Mostrar Todos' : 'Mostrar Solo Equipo' }}</button>
      </nav>
    </header>
    <div v-if="view === 'pokedex'">
      <type-filter :types="types" @update:selected-types="updateSelectedTypes"></type-filter>
      <range-slider :min="1" :max="151" :value="numberRange" @update:min="updateMinRange" @update:max="updateMaxRange"></range-slider>
      <div v-if="loaded && filteredPokemons.length > 0">
        <div class="pokedex-container">
          <pokemon-card v-for="pokemon in filteredPokemons" :key="pokemon.name" :pokemon="pokemon" :in-team="team.includes(pokemon)" @toggle-team="toggleTeam"/>
        </div>
      </div>
      <div v-else-if="loaded">
        <h2>No se han encontrado Pokémon que coincidan con los filtros aplicados.</h2>
      </div>
      <div v-else>
        <h2>Cargando Pokédex...</h2>
      </div>
      <div class="team-container">
        <h2>Team</h2>
        <img v-for="member in team" :key="member.id" :src="member.image" :alt="`Sprite de ${member.name}`"/>
      </div>
    </div>
    <div v-else-if="view === 'inventory'">
      <inventory :items="inventoryItems" />
    </div>
    <div v-else-if="view === 'store'">
      <store :items="storeItems" @update-inventory="updateInventory"/>
      <button class="buy-button" @click="buyItems">Comprar</button>
    </div>
  </div>
</template>


<script>
import axios from 'axios';
import PokemonCard from './components/pokeCard.vue';
import RangeSlider from './components/slider.vue';
import TypeFilter from './components/tipos.vue';
import Inventory from './components/inventario.vue';
import Store from './components/shop.vue';

export default {
  name: 'Pokedex',
  components: {
    PokemonCard,
    RangeSlider,
    TypeFilter,
    Inventory,
    Store
  },
  data() {
    return {
      view: 'pokedex',
      pokemons: [],
      team: [],
      loaded: false,
      selectedTypes: [],
      numberRange: [1, 151],
      types: ['fire', 'water', 'grass', 'electric', 'ice', 'fighting', 'poison', 'ground', 'flying', 'psychic', 'bug', 'rock', 'ghost', 'dragon', 'dark', 'steel', 'fairy'],
      showTeamOnly: false,
      showFavoritesOnly: false,
      favorites: [],
      inventoryItems: {
        pokeball: 10,
        greatball: 5,
        ultraball: 2,
        potion: 5,
        elixir: 3
      },
      storeItems: {
        pokeball: { name: 'Pokéball', icon: '', max: 99 },
        greatball: { name: 'Greatball', icon: '', max: 99 },
        ultraball: { name: 'Ultraball', icon: '', max: 99 },
        potion: { name: 'Poción', icon: '', max: 99 },
        elixir: { name: 'Elixir', icon: '', max: 99 }
      }
    };
  },
  computed: {
    filteredPokemons() {
      return this.pokemons.filter(pokemon => {
        const isInTeam = this.showTeamOnly ? this.team.some(teamMember => teamMember.id === pokemon.id) : true;
        const isFavorite = this.showFavoritesOnly ? this.favorites.includes(pokemon.id) : true;
        return (this.selectedTypes.length === 0 || pokemon.types.some(type => this.selectedTypes.includes(type.type.name))) &&
               (pokemon.id >= this.numberRange[0] && pokemon.id <= this.numberRange[1]) &&
               isInTeam && isFavorite;
      });
    }
  },
  methods: {
    toggleTeam(pokemon) {
      const index = this.team.indexOf(pokemon);
      if (index > -1) {
        this.team.splice(index, 1);
      } else if (this.team.length < 6) {
        this.team.push(pokemon);
      }
    },
    updateSelectedTypes(types) {
      this.selectedTypes = types;
    },
    updateMinRange(min) {
      this.numberRange[0] = min;
    },
    updateMaxRange(max) {
      this.numberRange[1] = max;
    },
    toggleTeamFilter() {
      this.showTeamOnly = !this.showTeamOnly;
    },
    toggleFavoriteFilter() {
      this.showFavoritesOnly = !this.showFavoritesOnly;
    },
    toggleFavorite(pokemon) {
      const index = this.favorites.indexOf(pokemon.id);
      if (index > -1) {
        this.favorites.splice(index, 1);
      } else {
        this.favorites.push(pokemon.id);
      }
    },
    updateInventory(item, quantity) {
      if (this.inventoryItems[item] + quantity <= this.storeItems[item].max) {
        this.inventoryItems[item] += quantity;
      } else {
        this.inventoryItems[item] = this.storeItems[item].max;
      }
    },
    goToPokedex() {
      this.view = 'pokedex';
    },
    goToInventory() {
      this.view = 'inventory';
    },
    goToStore() {
      this.view = 'store';
    },
    buyItems() {
      Object.keys(this.storeItems).forEach(item => {
        this.updateInventory(item, this.$refs.store.selectedItems[item]);
        this.$refs.store.resetSelection(item);
      });
    }
  },
  async mounted() {
    try {
      const response = await axios.get('https://pokeapi.co/api/v2/pokemon?limit=151');
      this.pokemons = await Promise.all(response.data.results.map(async pokemon => {
        const res = await axios.get(pokemon.url);
        return {
          id: res.data.id,
          name: res.data.name,
          image: res.data.sprites.front_default,
          types: res.data.types
        };
      }));
      this.loaded = true;

      const itemNames = {
        pokeball: 'poke-ball',
        greatball: 'great-ball',
        ultraball: 'ultra-ball',
        potion: 'potion',
        elixir: 'elixir'
      };
      const itemRequests = Object.keys(itemNames).map(name => axios.get(`https://pokeapi.co/api/v2/item/${itemNames[name]}`));
      const itemResponses = await Promise.all(itemRequests);
      itemResponses.forEach((response, index) => {
        const key = Object.keys(itemNames)[index];
        this.storeItems[key].icon = response.data.sprites.default;
      });
    } catch (error) {
      console.error("Error loading data: ", error);
      this.loaded = true;
    }
  }
}
</script>

<style>
body {
  display: block;
  place-items: center;
  background-color: #121212;
}

#app {
  display: contents;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 10px 20px;
  background-color: #C00000;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.app-header h1 {
  font-size: 24px;
  color: #fff;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}

.app-header nav button {
  background-color: #121212;
  color: whitesmoke;
  border: none;
  border-radius: 8px;
  padding: 10px 20px;
  margin: 0 5px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.app-header nav button:hover {
  background-color: #121212;
}

.pokedex-container {
  display: ruby;
  flex-wrap: wrap;
  justify-content: flex-start;
  align-items: flex-start;
  margin: 0 auto;
  width: 100%;
  padding: 10px 0;
}

.pokemon-card {
  flex: 1 1 21%;
  margin: 10px;
  box-sizing: border-box;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  min-width: 200px;
  max-width: 250px;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.3), rgba(255, 255, 255, 0));
  background-color: black;
  position: relative;
  overflow: visible;
  transition: background 0.7s ease, transform 0.3s ease;
}

.pokemon-card:hover {
  transform: scale(1.05);
  background: linear-gradient(to top left, rgba(255, 255, 255, 0.3), rgba(255, 255, 255, 0));
}

.pokemon-card::before {
  content: "";
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background-image: radial-gradient(circle, rgba(255, 255, 255, 0.25) 0%, transparent 60%);
  transform: rotate(0deg);
  transition: transform 0.7s ease-out;
}

.pokemon-card:hover::before {
  transform: rotate(180deg);
  filter: blur(6px);
}

.favorite-checkbox {
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 20px;
  height: 20px;
  border: 2px solid #ccc;
  border-radius: 50%;
  background-color: transparent;
  cursor: pointer;
  transition: background-color 0.3s ease;
  z-index: 2;
}

.favorite-checkbox.checked {
  background-color: #40E0D0;
  border-color: #40E0D0;
}

.favorite-checkbox:before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: white;
}

.team-container {
    position: fixed;
    right: 1%;
    top: 190px;
    display: flex;
    flex-direction: column;
    align-items: center;
    color: whitesmoke;
    background-color: #3a3a3a;
    border-radius: 10px;
}

.team-container img {
  width: 100px;
  height: 100px;
  margin: 5px;
}

.inventory-container, .store-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: flex-start;
  margin: 0 auto;
  width: 100%;
  padding: 10px 0;
}

.inventory-card, .store-card {
  flex: 1 1 21%;
  margin: 10px;
  box-sizing: border-box;
  text-align: center;
  border: 1px solid #ccc;
  color: whitesmoke;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  min-width: 200px;
  max-width: 250px;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(5px);
  transition: transform 0.3s ease;
}

.store-card img {
  width: 100px; /* Incremento del tamaño de la imagen */
  height: 100px; /* Incremento del tamaño de la imagen */
  object-fit: contain;
  margin-bottom: 10px;
}

.inventory-card:hover, .store-card:hover {
  transform: scale(1.05);
}

.item-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.item-name {
  font-size: 14px;
  font-weight: bold;
  margin: 5px;
  position: absolute;
  top: 5px;
  left: 5px;
}

.item-controls {
  display: flex;
  align-items: center;
  justify-content: center;
}

.control-symbol {
  cursor: pointer;
  font-size: 16px;
  margin: 0 5px;
}

.item-quantity {
  font-size: 14px;
  font-weight: bold;
  position: absolute;
  bottom: 5px;
  right: 5px;
  background: rgba(0, 0, 0, 0.5);
  color: #fff;
  border-radius: 50%;
  padding: 5px 10px;
  font-size: 12px;
}

.buy-button {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #C00000;
  color: whitesmoke;
  border: none;
  border-radius: 8px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
}

.buy-button:hover {
  background-color: #121212;
}
</style>




