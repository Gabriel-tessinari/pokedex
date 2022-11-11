<script setup lang="ts">
import { PropType, ref } from 'vue';
import axios from 'axios';

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

class Page {
  page: number;
  isActive: boolean;
  pokemon: Pokemon[];

  constructor() {
    this.page = 0;
    this.isActive = false;
    this.pokemon = [];
  }
}

const props = defineProps({
  gen: Object as PropType<Generation>
});

const emit = defineEmits(['select']);

const pokemonPerPage = 15;
const fullPokeList = ref<Pokemon[]>([]);
const pokeList = ref<Pokemon[]>([]);
const pages = ref<Page[]>([]);

async function  setFullList() {
  let limit: number = 0;
  let offset: number = 0;

  if(props.gen) {
    offset = props.gen.from - 1;
    limit = props.gen.to - offset;
  }

  const url = 'https://pokeapi.co/api/v2/pokemon?limit=' + limit + '&offset=' + offset;
  
  await axios.get(url)
  .then(response => fullPokeList.value = response.data.results)
  .catch(() => fullPokeList.value = []);

  setPages();
}

function setPages() {
  pages.value = [];
  const totalPages = Math.ceil(fullPokeList.value.length / pokemonPerPage);

  for(let i = 1; i <= totalPages; i++) {
    let page = new Page();
    page.page = i;
    page.isActive = (i == 1);
    setPageList(page);
    i == 1 ? pokeList.value = page.pokemon : '';
    pages.value.push(page);
  }
}

function setPageList(page: Page) {
  let min = pokemonPerPage * (page.page - 1);
  let max = min + pokemonPerPage;
  max > fullPokeList.value.length ? max = fullPokeList.value.length : '';
  let auxList: Pokemon[] = [];

  for(let i = min; i < max; i++) {
    auxList.push(fullPokeList.value[i]);
  }

  page.pokemon = auxList;
}

function selectPage(index: number) {
  pages.value.forEach((page, idx) => {
    page.isActive = false;

    if(idx == index) {
      page.isActive = true;
      pokeList.value = page.pokemon;
    }
  });
}

function selectPokemon(pokemon: Pokemon) {
  emit('select', pokemon);
}
</script>

<template>
  <div class="box">
    <button class="button is-warning is-fullwidth mb-2" @click="setFullList()">
      Show Gen {{gen?.id}}
    </button>
    <p v-for="item in pokeList" :key="item.name" @click="selectPokemon(item)">
      {{item.name.toUpperCase()}}
    </p>
    <nav class="pagination is-small mt-4" role="navigation" aria-label="pagination"
    v-if="pages.length > 0">
      <ul class="pagination-list">
        <li v-for="(item, index) in pages" :key="item.page"
        @click="selectPage(index)">
          <a class="pagination-link" :class="item.isActive ? 'is-current' : ''">
            {{item.page}}
          </a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<style scoped lang="scss">
@import '../node_modules/bulma/bulma.sass';

p {
  text-align: center;
  font-size: 1.2rem;
  font-weight: 700;
  border: 1px solid $black;
  border-radius: .25rem;
  margin: 2px;
  padding: .25rem;

  &:hover {
    background-color: $yellow;
    border-color: $blue;
    color: $blue;
    cursor: pointer;
  }
}

.pagination-list {
  justify-content: center !important;

  .pagination-link {
    text-decoration: none !important;

    &.is-current {
      color: $white !important;
    }
  }
}
</style>