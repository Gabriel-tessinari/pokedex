<script setup lang="ts">
import { ref } from 'vue';
import PokeList from './components/PokeList.vue';
import PokeInfo from './components/PokeInfo.vue';

class Pokemon {
  name: string;
  url: string;

  constructor() {
    this.name = '';
    this.url = '';
  }
}

class Generation {
  id: number;
  region: string;
  from: number;
  to: number;

  constructor(id: number, region: string, from: number, to: number) {
    this.id = id;
    this.region = region;
    this.from = from;
    this.to = to;
  }
}

const generations = ref<Generation[]>([
  new Generation(1, 'Kanto', 1, 151),
  new Generation(2, 'Johto', 152, 251),
  new Generation(3, 'Hoenn', 252, 386),
  new Generation(4, 'Sinnoh', 387, 493),
  new Generation(5, 'Unova', 494, 649),
  new Generation(6, 'Kalos', 650, 721),
  new Generation(7, 'Alola', 722, 809),
  new Generation(8, 'Galar/Hisui', 810, 905)
]);

const gen = ref<Generation>(generations.value[0]);
const pokemon = ref<Pokemon>(new Pokemon());

function previous() {
  let previousGen: number = gen.value.id - 1;
  if(previousGen == 0) previousGen = 8;
  
  const previous = generations.value.find(item => item.id == previousGen);
  if(previous) gen.value = previous;
}

function next() {
  let nextGen: number = gen.value.id + 1;
  if(nextGen == 9) nextGen = 1;
  
  const next = generations.value.find(item => item.id == nextGen);
  if(next) gen.value = next;
}
</script>

<template>
  <div class="tile is-ancestor">
    <div class="tile is-parent is-vertical is-4">
      <div class="tile is-child notification is-danger">
        <div class="columns header">
          <div class="column is-2">
            <button class="button is-info is-fullwidth" @click="previous()">
              <span class="material-symbols-outlined">arrow_back</span>
            </button>
          </div>
          <div class="column title is-8 m-0">
            <p>{{gen.region}}</p>
          </div>
          <div class="column is-2">
            <button class="button is-info is-fullwidth" @click="next()">
              <span class="material-symbols-outlined">arrow_forward</span>
            </button>
          </div>
        </div>
        <PokeList :gen="gen" @select="(response) => pokemon = response"></PokeList>
      </div>
    </div>
    <div class="tile is-parent is-vertical is-8" v-if="pokemon.name">
      <div class="tile is-child notification is-link">
        <p class="mb-4">{{pokemon.name.toUpperCase()}}</p>
        <PokeInfo :pokemon="pokemon"></PokeInfo>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.tile {
  &> p {
    font-weight: 700;
    font-size: 1.5rem;
  }
  .columns.header {
    display: flex !important;
    flex-direction: row !important;
    flex-wrap: nowrap !important;
    align-items: center !important;

    .column.title {
      display: flex;
      justify-content: center;

      &> p {
        font-weight: 700;
        font-size: 1.5rem;
      }
    }
  }
}
</style>
