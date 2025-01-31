<template>
    <div class="p-4 flex items-center justify-center flex-col bg-gray-900 min-h-screen">
        <h1 class="text-3xl font-bold mb-6 text-white">Movie Search</h1>
        <input v-model="searchQuery" @input="searchMovies" placeholder="Search for a movie..."
            class="p-4 border-2 border-gray-500 rounded-full max-w-lg w-full mb-6 text-lg focus:outline-none focus:ring-2 focus:ring-blue-400 transition duration-300 text-white" />
        <div v-if="loading" class="text-center text-white">Loading...</div>
        <div v-else
            class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6 w-full max-w-6xl">
            <div v-for="movie in movies" :key="movie.imdbID"
                class="mb-4 p-4 shadow-lg bg-gray-800 text-white rounded-lg hover:scale-105 hover:shadow-2xl transition duration-300"
                @click="fetchMovieDetails(movie.imdbID)"> 
                <img :src="movie.Poster" alt="Movie Poster" class="w-full h-48 object-cover rounded-mb mb-4" />
                <h2 class="text-xl font-semibold truncate">{{ movie.Title }}</h2>
                <p class="text-sm text-gray-400">{{ movie.Year }}</p>
            </div>
        </div>  
        <div v-if="selectedMovie" class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center">
          <div class="bg-white text-black p-8 rounded-lg w-full max-w-xl">
              <h2 class="text-2xl font-bold">{{ selectedMovie.Title }}</h2>
              <p>{{ selectedMovie.Plot }}</p>
              <p><strong>Director:</strong> {{ selectedMovie.Director }}</p>
              <p><strong>Actors:</strong> {{ selectedMovie.Actors }}</p>
              <p><strong>Rating:</strong> {{ selectedMovie.imdbRating }}</p>
              <button @click="selectedMovie = null" class="mt-4 p-2 bg-blue-500 text-white rounded-full hover:scale-105 cursor-pointer transition duration-300">Close</button>
          </div>
      </div>
    </div>
</template>

<script setup>
import { ref, watch } from 'vue';
import { debounce } from 'lodash';

const searchQuery = ref('');
const movies = ref([]);
const loading = ref(false);
const selectedMovie = ref(null);

const searchMovies = debounce(async () => {
    if (searchQuery.value.length < 3) {
        return;
    }
    loading.value = true;
    try {
        const response = await fetch(`https://www.omdbapi.com/?s=${searchQuery.value}&apikey=${import.meta.env.VITE_OMDB_API_KEY}`);
        const data = await response.json();
        movies.value = data.Search || [];
    } catch (error) {
        console.error("Error fetching movies", error);
    } finally {
        loading.value = false;
    }
}, 500);

watch(searchQuery, searchMovies);

const fetchMovieDetails = async (imdbID) => {
    try {
        const apiKey = import.meta.env.VITE_OMDB_API_KEY;
        const url = `https://www.omdbapi.com/?i=${imdbID}&apikey=${apiKey}`;
        const response = await fetch(url);
        const data = await response.json();
        selectedMovie.value = data; 
    } catch (error) {
        console.error('Error fetching movie details:', error);
    }
};
</script>
