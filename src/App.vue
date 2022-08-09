<script setup>
import {ref, computed, onMounted} from "vue"

const query = ref('');
const my_anime_list = ref([])
const search_results = ref([])

const my_anime_list_asc = computed(()=>{
  return my_anime_list.value.sort((a, b)=>{
    return a.title.localeCompare(b.title)
  })
})
const searchAnime = ()=>{
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`
  fetch(url)
      .then(res => res.json())
      .then(res =>{
        search_results.value = res.data
      })
}

const handleInput= (e) =>{
  if(!e.target.value){
    search_results.value = []
  }
}
const addAnime = (anime) =>{
  search_results.value = []
  query.value = ''

  my_anime_list.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0
  })
  localStorage.setItem('my_anime_list', JSON.stringify(my_anime_list.value))
}

const increaseWatch = (anime) =>{
  anime.watched_episodes++
  localStorage.setItem('my_anime_list', JSON.stringify(my_anime_list.value))

}
const decreaseWatch = (anime) =>{
  anime.watched_episodes--
  localStorage.setItem('my_anime_list', JSON.stringify(my_anime_list.value))

}

onMounted(()=>{
  my_anime_list.value = JSON.parse(localStorage.getItem('my_anime_list')) || []
})
</script>

<template>
<main class="container flex justify-center items-center flex-col my-7 min-w-full">
  <h1 class="text-xl font-semibold mb-8">Anime Tracker</h1>
  <form v-on:submit.prevent="searchAnime">
  <input class="placeholder:italic placeholder:text-slate-400  bg-white  border border-slate-300 rounded-md py-2 pl-2 pr-3 shadow-sm focus:outline-none focus:border-sky-500 focus:ring-sky-500 focus:ring-1 sm:text-sm" type="text" placeholder="search anime" name="" id="" v-model="query" @input="handleInput"/>
  <button type="submit" class="ml-5 bg-blue-500 py-2 px-6 rounded-lg text-white text-medium">Search</button>
  </form>

  <div class="results w-1/2 bg-red-500 my-8 py-8 px-9 flex flex-col items-center " v-if="search_results.length > 0">
  <div class="result flex flex-row space-x-4 space-y-5 mb-5" v-for="anime in search_results" :key="anime.id">
      <img class="w-24" :src="anime.images.jpg.image_url" alt="" srcset="">
      <div class="details">
        <h3 class="text-lg font-bold">{{ anime.title }}</h3>
        <p class="text-sm" :title="anime.synopsis" v-if="anime.synopsis">
        {{ anime.synopsis.slice(0, 100) }}...
        </p>
        <span>
        </span>
        <button class="mt-4 bg-green-600 py-2 px-4 rounded-lg text-white text-medium" @click="addAnime(anime)">Add to my list</button>
      </div>
  </div>
  </div>

  <div class="myanime my-7" v-if="my_anime_list.length > 0">
  <h2 class="text-xl font-semibold mb-8">My List Anime</h2>

  <div class="anime flex flex-row" v-for="anime in my_anime_list_asc" :key="anime.id">
  <img class="w-24" :src="anime.image" />
  <div class="px-4 flex flex-col w-1/2 ">
     <h3 class="text-lg font-bold">{{ anime.title }}</h3>
  <span class="episodes text-sm ">{{ anime.watched_episodes }} / {{ anime.total_episodes }}</span>

<button class="bg-blue-500 mb-4" v-if="anime.total_episodes !== anime.watched_episodes" @click="increaseWatch(anime)">+</button>
<button class="bg-red-600" v-if="anime.watched_episodes > 0" @click="decreaseWatch(anime)">-</button>
  </div>
  </div>
  </div>
</main>
</template>

<style scoped>
</style>
