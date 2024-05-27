<template>
  <div class="pokemon-card" :class="{ 'selected': inTeam }" @click="toggleInTeam">
    <div class="card-background" :style="cardStyle"></div>
    <div class="pokemon-content">
      <div class="pokemon-header">
        <h3 class="pokemon-name">{{ pokemon.name }}</h3>
        <span class="pokemon-number">Nº {{ pokemon.id }}</span>
      </div>
      <img :src="pokemon.image" :alt="`Imagen de ${pokemon.name}`" />
      <div class="pokemon-types">
        <span v-for="type in pokemon.types" :key="type.type.name" class="pokemon-type" :style="{ backgroundColor: getTypeColor(type.type.name) }">
          {{ type.type.name }}
        </span>
      </div>
      <div class="favorite-checkbox" :class="{ 'checked': isFavorite }" @click.stop="toggleFavorite"></div>
    </div>
  </div>
</template>



<script>
export default {
  name: 'PokemonCard',
  props: ['pokemon', 'inTeam'],
  data() {
    return {
      isFavorite: false,
      typeColors: {
        fire: '#F08030',
        water: '#6890F0',
        grass: '#78C850',
        electric: '#F8D030',
        psychic: '#F85888',
        ice: '#98D8D8',
        dragon: '#7038F8',
        dark: '#705848',
        fairy: '#EE99AC',
        normal: '#A8A878',
        fighting: '#C03028',
        flying: '#A890F0',
        poison: '#A040A0',
        ground: '#E0C068',
        rock: '#B8A038',
        bug: '#A8B820',
        ghost: '#705898',
        steel: '#B8B8D0'
      }
    };
  },
  computed: {
    cardStyle() {
      if (this.pokemon.types.length > 1) {
        return {
          backgroundImage: `linear-gradient(to bottom, ${this.getTypeColor(this.pokemon.types[0].type.name)}, ${this.getTypeColor(this.pokemon.types[1].type.name)})`
        };
      } else {
        return {
          backgroundColor: this.getTypeColor(this.pokemon.types[0].type.name)
        };
      }
    }
  },
  methods: {
    toggleInTeam() {
      this.$emit('toggle-team', this.pokemon);
    },
    toggleFavorite() {
      this.isFavorite = !this.isFavorite;
      this.$emit('toggle-favorite', this.pokemon);
    },
    getTypeColor(type) {
      return this.typeColors[type] || '#68A090';
    }
  }
}
</script>



<style scoped>
.pokemon-card {
  position: relative;
  border: 4px solid black;
  border-radius: 12px;
  margin: 8px 8px 30px;
  width: 220px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  transition: transform 0.3s ease;
  overflow: hidden;
}

.card-background {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 0;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.3), rgba(255, 255, 255, 0));
  background-color: black;
}

.pokemon-content {
  position: relative;
  z-index: 1;
  width: 100%;
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

.pokemon-card img {
  width: 100%;
  height: auto;
  border-radius: 4px;
  margin-top: 10px;
}

.pokemon-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.pokemon-name, .pokemon-number {
  font-size: 14px;
  font-weight: bold;
}

.pokemon-types {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin-bottom: 15px;
}

.pokemon-type {
  background-color: #eee;
  border-radius: 10px;
  padding: 5px 10px;
  margin: 2px;
  width: 80px;
  text-align: center;
}

.favorite-star {
  position: absolute;
  bottom: 10px;
  right: 10px;
  cursor: pointer;
  color: gold;
  font-size: 24px;
}

.selected {
  background-color: rgba(255, 255, 255, 0.5);
}
</style>
