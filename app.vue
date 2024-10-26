<script setup lang="ts">
const id = ref(null);
const selectedGenres = ref([]);
const sortBy = ref("popularity.desc");
const headers = {
  Authorization:
    "Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI1ZTQxMWE4YjVhYzI3M2RhNDI0ODZhZGRiMDQ1YjY4OSIsIm5iZiI6MTcyOTk0OTE2MS43ODc2MzYsInN1YiI6IjVmNzcyMGU4MjE2MjFiMDAzNTNhMDJjYyIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.85FcQ1-dap9nr6dZH5ufF8Q4mLRBIRQZrE0GVw0nWmA",
  Accept: "application/json",
};
const { data: genres } = await useFetch(
  "https://api.themoviedb.org/3/genre/movie/list?language=en",
  {
    // @ts-ignore
    headers,
    lazy: true,
    server: false,
  }
);

const selectedGenresString = computed(() => selectedGenres.value.join("%2C"));
const url = computed(() => {
  return `https://api.themoviedb.org/3/discover/movie?include_adult=false&include_video=false&language=en-US&page=1&sort_by=${sortBy.value}&with_genres=${selectedGenresString.value}`;
});

const { data, status } = await useFetch(url, {
  // @ts-ignore
  headers,
  lazy: true,
  server: false,
  watch: [selectedGenres],
});
</script>

<template>
  <div class="bg-red-400">
    {{ id }}
    <br />
    <!-- <input v-model="id" /> -->
    {{ selectedGenres }}
    {{ selectedGenresString }}

    <div v-if="status === 'success'" class="flex">
      <div class="w-1/4">
        <div>
          sort by
          <select v-model="sortBy">
            <option value="popularity.asc">Popularity Asc</option>
            <option value="popularity.desc">Popularity Desc</option>
            <option value="release_date.asc">Release Date Asc</option>
            <option value="release_date.desc">Release Date Desc</option>
            <option value="vote_average.asc">Rating Asc</option>
            <option value="vote_average.desc">Rating Desc</option>
          </select>
        </div>
        <div>
          Genres:
          <div
            v-for="item in genres.genres"
            :key="item.id"
            class="flex justify-between w-20"
          >
            {{ item.name }}
            <input type="checkbox" v-model="selectedGenres" :value="item.id" />
          </div>
        </div>
      </div>
      <div class="grid grid-cols-4">
        <div v-for="item in data.results" :key="item.id" class="text-center">
          <!-- https://image.tmdb.org/t/p/w220_and_h330_face/cRDJxdnRb7ikKd6fVJTrGeaL34v.jpg -->
          <img
            :src="`https://image.tmdb.org/t/p/w220_and_h330_face/${item.poster_path}`"
            alt=""
          />
          {{ item.title }}
        </div>
      </div>
    </div>
    <div v-else-if="status === 'error'" class="">Error</div>
    <div v-else class="">Loading...</div>
  </div>
</template>
