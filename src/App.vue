<template>
  <main>
    <h1>Поиск аниме по названию</h1>
    <form action="#" @submit.prevent="searchAnime">
      <input
          type="text"
          placeholder="Например, naruto"
          v-model="query"
          @input="handleInput"
      />
      <button type="submit">Найти</button>
    </form>
    <div class="results" v-if="search_results.length>0">
      <div class="result" v-for="anime in search_results" :key="anime.mal_id">
        <img :src="anime.images.jpg.image_url" :alt="anime.title">
        <div class="details">
          <h3>{{ anime.title }}</h3>
          <p :class="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}...
          </p>
          <span class="flex-1"></span>
          <button class="add" @click="addAnime(anime)">Добавить</button>
        </div>
      </div>
    </div>
    <h3 class="not-found" v-if="!is_search_result">Ничего не найдено</h3>
    <div class="myanime" v-if="my_anime.length>0">
      <h2>Список сохраненных аниме</h2>
      <div class="anime" v-for="anime in my_anime_asc" :key="anime.id">
        <img :src="anime.image" :alt="anime.title">
        <h3>{{ anime.title }}</h3>
        <div class="flex-1"></div>
        <span class="episodes">
          {{ anime.watched_episodes }} / {{ anime.total_episodes }}
        </span>
        <button
            v-if="anime.total_episodes !== anime.watched_episodes"
            @click="increaseWatch(anime)"
            class="inc"
        >+
        </button>
        <button
            v-if="anime.watched_episodes > 0"
            @click="decreaseWatch(anime)"
            class="deg"
        >-
        </button>
        <button
            @click="deleteWatch(anime)"
            class="del"
        >×</button>
      </div>
    </div>
  </main>
</template>

<script setup>
import {ref, computed, onMounted} from 'vue'
import axios from 'axios'

const query = ref('')
const my_anime = ref([])
const search_results = ref([])
const is_search_result = ref(true);

const my_anime_asc = computed(() => {
  return my_anime.value.sort((a, b) => {
    return a.title.localeCompare(b.title)
  })
})

const searchAnime = async () => {
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`
  await axios.get(url)
      .then(res => search_results.value = res.data.data)
  is_search_result.value = search_results.value.length > 0 ? true : false;
}

const handleInput = e => {
  if (e.target.value) {
    search_results.value = []
  }
}

const addAnime = anime => {
  search_results.value = []
  query.value = ''
  is_search_result.value = true;

  my_anime.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0
  })

  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const increaseWatch = anime => {
  anime.watched_episodes++
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const decreaseWatch = anime => {
  anime.watched_episodes--
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const deleteWatch = anime => {
  my_anime.value = my_anime.value.filter(a=>a !== anime);
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

onMounted(() => {
  my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || []
})
</script>


<style>

@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  font-family: 'Roboto', 'Arial', sans-serif;
}

body {
  background: #eee;
}

main {
  margin: 0 auto;
  max-width: 768px;
  padding: 1.5rem;
}

h1 {
  text-align: center;
  margin-bottom: 1.5rem;
}
h3.not-found {
  text-align: center;
  margin-bottom: 1.5rem;
  color: #666;

}

form {
  display: flex;
  max-width: 480px;
  margin: 0 auto 1.5rem;
}

form input {
  appearance: auto;
  outline: none;
  border:none;

  background: #fff;
  color: #888;
  font-size: 1.125rem;
  padding: 0.5rem 1rem;
  width: 100%;
}

button {
  display: block;
  appearance: auto;
  outline: none;
  border:none;
  cursor: pointer;
  background: none;
  font-size: 1.125rem;
  text-transform: uppercase;
  padding: 0.5rem 1rem;
  background-image: linear-gradient(to right, deeppink 50%, darkviolet 50%);
  background-size: 200%;
  color: #fff;
  transition: 0.4s;
}
button:hover {
  background-position: right;
}

button.del {
  background-image: linear-gradient(to right, #ef9797 50%, #d70000 50%);
}

.results {
  background: #fff;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0,0,0,.1);
  max-height: 480px;
  overflow-y: scroll;
  margin-bottom: 1.5rem;
}

.result {
  display: flex;
  margin:1rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 0.5rem;
  transition: 0.4s;

}

.result img {
  width: 100px;
  border-radius: 1rem;
  margin-right: 1rem;
}

.result .details {
  flex:1 1 0%;
  display: flex;
  align-items: flex-start;
  flex-direction: column;
}

.flex-1 {
  flex: 1 1 0%;
}

.details h3 {
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
}

.details p {
  font-size: 0.875rem;
  margin-bottom: 0.5rem;
  line-height: 1.4;
}

.details button {
  margin-left: auto;
}

.myanime h2 {
  color: #888;
  font-weight: 400;
  margin-bottom: 1.5rem;
}

.myanime .anime {
  display: flex;
  align-items: center;
  margin-bottom: 1.5rem;
  background: #fff;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0,0,0,.1);
}

.anime img {
  width: 72px;
  height: 72px;
  object-fit: cover;
  border-radius: 1rem;
  margin-right: 1rem;
}

.anime h3 {
  color: #888;
  font-size: 1.125rem;
}

.anime .episodes {
  margin-right: 1rem;
  color: #888;
  white-space: nowrap;
}

.anime button {
  margin-left: 1rem;
}
.anime button:last-of-type {
  margin-right: 0;
}

</style>
