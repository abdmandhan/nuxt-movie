<script setup lang="ts">
const search = ref(null);
const debouncedSearch = ref("");

const selectedGenres = ref([]);
const sortBy = ref("popularity.desc");
const page = ref(1);
const productsFetched: any = ref([]);
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
  return `https://api.themoviedb.org/3/discover/movie?include_adult=false&include_video=false&language=en-US&page=${page.value}&sort_by=${sortBy.value}&with_genres=${selectedGenresString.value}`;
});

const { data, status } = await useFetch(url, {
  // @ts-ignore
  headers,
  lazy: true,
  server: false,
  watch: [selectedGenres, sortBy, page],
  transform: (res: any) => {
    if (page.value === 1) {
      productsFetched.value = res.results;
    } else {
      productsFetched.value = [...productsFetched.value, ...res.results];
    }
    return productsFetched.value;
  },
});
function debounce(fn: Function, delay: number) {
  // @ts-ignore
  let timeoutId: NodeJS.Timeout;
  return function (...args: any[]) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

watch(
  search,
  debounce((newValue: string) => {
    debouncedSearch.value = newValue;
  }, 1000)
);

const searchUrl = computed(() => {
  return `https://api.themoviedb.org/3/search/movie?query=${search.value}&include_adult=false&language=en-US&page=1`;
});

const { data: searchResult } = await useFetch(searchUrl, {
  // @ts-ignore
  headers,
  lazy: true,
  server: false,
  watch: [debouncedSearch],
});
</script>

<template>
  <div class="bg-red-400">
    {{ search }}
    <br />
    <input v-model="search" />
    <div>
      <ul>
        <li v-for="item in searchResult" :key="item.id">
          {{ item.title }}
        </li>
      </ul>
    </div>
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
            class="flex justify-between px-2 hover:bg-gray-200"
          >
            {{ item.name }}
            <input type="checkbox" v-model="selectedGenres" :value="item.id" />
          </div>
        </div>
      </div>
      <div class="grid grid-cols-4">
        <div v-for="item in productsFetched" :key="item.id" class="text-center">
          <img
            :src="`https://image.tmdb.org/t/p/w220_and_h330_face/${item.poster_path}`"
            alt=""
          />
          {{ item.title }}
        </div>
      </div>
      <button @click="page++">load more</button>
    </div>
    <div v-else-if="status === 'error'" class="">Error</div>
    <div v-else class="">Loading...</div>
  </div>
</template>
