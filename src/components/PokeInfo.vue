<script setup lang="ts">
import axios from 'axios';
import { onMounted, PropType, ref, watch } from 'vue';

class Pokemon {
  name: string;
  url: string;
  id?: number;
  image?: string;
  height?: number;
  weight?: number;
  types?: [
    {type: {name: string}}
  ];
  stats?: [
    {
      base_stat: number
      stat: {name: string}
    }
  ];

  constructor() {
    this.name = '';
    this.url = '';
  }
}

const props = defineProps({
  pokemon: Object as PropType<Pokemon> 
});

const pokemon = ref<Pokemon>(new Pokemon());

async function loadPokemon() {
  if(props.pokemon) {
    await axios.get(props.pokemon.url)
    .then(response => {
      pokemon.value = response.data;
      pokemon.value.height = response.data.height/10;
      pokemon.value.weight = response.data.weight/10;
      setImage(pokemon.value);
      setStatName(pokemon.value);
    })
    .catch(error => console.log(error));
  }
}

function setImage(pokemon: Pokemon) {
  let id: string = '';

  if(pokemon.id && pokemon.id < 10) id = '00' + pokemon.id;
  else if(pokemon.id && pokemon.id < 100) id = '0' + pokemon.id;
  else id += pokemon.id;

  pokemon.image = 'https://assets.pokemon.com/assets/cms2/img/pokedex/full/' + id + '.png';
}

function setStatName(pokemon: Pokemon) {
  if(pokemon.stats) {
    pokemon.stats.forEach(item => {
      if(item.stat.name == 'hp') item.stat.name = 'HP';
      else if(item.stat.name == 'attack') item.stat.name = 'Attack';
      else if(item.stat.name == 'defense') item.stat.name = 'Defense';
      else if(item.stat.name == 'special-attack') item.stat.name = 'Sp. Attack';
      else if(item.stat.name == 'special-defense') item.stat.name = 'Sp. Defense';
      else if(item.stat.name == 'speed') item.stat.name = 'Speed';
    });
  }
}

watch(() => props.pokemon, () => {
  loadPokemon()
});

onMounted(() => {
  loadPokemon()
});
</script>

<template>
  <div class="box">
    <div class="columns pokemon-header">
      <div class="column is-2-mobile">
        <p class="pokemon-id">#{{pokemon.id}}</p>
      </div>
      <div class="column is-10-mobile pokemon-types">
          <figure v-for="(item, index) in pokemon.types" :key="index" 
          class="image is-48x48 mx-1">
            <img class="is-rounded" :src="'/src/assets/' + item.type.name + '.png'" :alt="item.type.name">
          </figure>
      </div>
    </div>
    <div class="columns">
      <div class="column is-6">
        <figure class="image is-1by1">
          <img :src="pokemon.image" :alt="pokemon.name">
        </figure>
      </div>
      <div class="column is-6 pokemon-infos">
        <section class="infos mb-4">
          <div class="columns">
            <div class="column is-6">
              <p><strong>Height:</strong> {{pokemon.height?.toFixed(2)}} m</p>
            </div>
            <div class="column is-6">
              <p><strong>Weight:</strong> {{pokemon.weight?.toFixed(2)}} kg</p>
            </div>
          </div>
        </section>
        <section class="stats">
          <p v-for="(item, index) in pokemon.stats" :key="index">
            <strong>{{item.stat.name}}:</strong> {{item.base_stat}}
            <progress class="progress is-info mt-1" :value="item.base_stat" max="300">15%</progress>
          </p>
        </section>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
@import '../node_modules/bulma/bulma.sass';

.pokemon-header {
  display: flex !important;
}
.pokemon-id {
  font-size: 2rem;
  font-weight: 700;
}

.pokemon-types {
  display: flex !important;
  flex-direction: row !important;
  justify-content: flex-end;

  p {
    font-size: 2rem;
    font-weight: 700;
    margin: 0 .5rem;
  }
}

.pokemon-infos {
  p {
    font-size: 1.5rem;
  }

  section.infos {
    background-color: $yellow;
    border-radius: .25rem;
    padding: .5rem;
  }

  section.stats {
    background-color: $green;
    border-radius: .25rem;
    padding: .75rem;
  }
}
</style>