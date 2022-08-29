<template>
  <div class="home">
    <!-- Hero -->
    <Hero />

    <!-- Search -->
    <div class="container search">
      <!-- v-model directive creates a two-way data binding between a value in the template and a value in data properties. -->
      <!-- .lazy modifier changes the v-model so it only syncs after a change -->
      <input
        v-model.lazy="searchInput"
        type="text"
        placeholder="Search"
        @keyup.enter="$fetch"
      />
      <button v-show="searchInput !== ''" class="button" @click="clearSearch">
        Clear Search
      </button>
    </div>

    <!-- Loading -->
    <Loading v-if="$fetchState.pending" />

    <!-- Movies -->
    <div v-else class="container movies">
      <!-- Searched Movies -->
      <div v-if="searchInput !== ''" id="movie-grid" class="movies-grid">
        <div
          v-for="(movie, index) in searchedMovies"
          :key="index"
          class="movie"
        >
          <div class="movie-img">
            <!-- the poster_path and other parameter are gotten from the data which can be found in the movie section of the api doc. -->
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
              alt="movie poster"
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title.slice(0, 25) }}
              <span v-if="movie.title.length > 25">...</span>
            </p>

            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>

            <!-- nuxt-link works like a vue router-link, when clicked, it takes the user to a different page. -->
            <!-- the name = name specified in the nuxt router/ movie pages folder & file name -->
            <!-- the params is gotten from the router and initialized to the movie id gotten from the data -->
            <nuxt-link
              class="button button-light"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
              >Get More Info</nuxt-link
            >
          </div>
        </div>
      </div>
      <!-- end of first movies-grid div -->

      <!-- Now Playing Movies -->
      <div v-else id="movie-grid" class="movies-grid">
        <div v-for="(movie, index) in movies" :key="index" class="movie">
          <div class="movie-img">
            <!-- the poster_path and other parameter are gotten from the data which can be found in the movie section of the api doc. -->
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
              alt="movie poster"
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title.slice(0, 25) }}
              <span v-if="movie.title.length > 25">...</span>
            </p>

            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>

            <!-- nuxt-link works like a vue router-link, when clicked, it takes the user to a different page. -->
            <!-- the name = name specified in the nuxt router/ movie pages folder & file name -->
            <!-- the params is gotten from the router and initialized to the movie id gotten from the data -->
            <nuxt-link
              class="button button-light"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
              >Get More Info</nuxt-link
            >
          </div>
        </div>
      </div>
      <!-- end of second movies-grid div -->
    </div>
    <!-- end of movie container -->
  </div>
  <!-- end of home div -->
</template>

<script>
import axios from 'axios'
import Loading from '../components/Loading.vue'

export default {
  components: { Loading },
  data() {
    return {
      movies: [],
      searchedMovies: [],
      searchInput: '',
    }
  },
  // the fetch hook is for fetching data asynchronously
  // it is called on server-side for rendering the route and on client-side when navigating.
  async fetch() {
    this.searchInput === '' ? await this.getMovies() : await this.searchMovies()
  },
  fetchDelay: 1000,
  // Search Engine Optimization(SEO)
  head() {
    return {
      title: 'Movie Website - Latest Streaming Movies',
      meta: [
        {
          // unique identifier
          hid: 'description',
          name: 'description',
          content: 'Get all the latest streaming movies in theaters & online',
        },
        {
          // unique identifier
          hid: 'keywords',
          name: 'keywords',
          content: 'movies, stream, streaming',
        },
      ],
    }
  },
  methods: {
    async getMovies() {
      const apiKey = process.env.API_KEY
      const movieUrl = `https://api.themoviedb.org/3/movie/now_playing?api_key=${apiKey}&language=en-Us&page=1`
      const data = axios.get(movieUrl)
      const result = await data
      // 'result' here is gotten from the data
      result.data.results.forEach((movie) => {
        this.movies.push(movie)
      })
      // console.log(this.movies)
    },
    async searchMovies() {
      const apiKey = process.env.API_KEY
      const searchUrl = `https://api.themoviedb.org/3/search/movie?api_key=${apiKey}&language=en-US&page=1&query=${this.searchInput}`
      const data = axios.get(searchUrl)
      const result = await data
      result.data.results.forEach((movie) => {
        this.searchedMovies.push(movie)
      })
      // console.log(this.searchedMovies)
    },
    clearSearch() {
      this.searchInput = ''
      this.searchMovies = []
    },
  },
}
</script>

<style lang="scss">
.home {
  .loading {
    padding-top: 120px;
    align-items: flex-start;
  }
  .search {
    display: flex;
    padding: 32px 16px;
    input {
      max-width: 350px;
      width: 100%;
      padding: 12px 6px;
      font-size: 14px;
      border: none;
      &:focus {
        outline: none;
      }
    }
    .button {
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;
    }
  }
  .movies {
    padding: 32px 16px;

    .movies-grid {
      display: grid;
      column-gap: 32px;
      row-gap: 64px;
      grid-template-columns: 1fr;
      @media (min-width: 500px) {
        grid-template-columns: repeat(2, 1fr);
      }
      @media (min-width: 750px) {
        grid-template-columns: repeat(3, 1fr);
      }
      @media (min-width: 1100px) {
        grid-template-columns: repeat(4, 1fr);
      }
      .movie {
        position: relative;
        display: flex;
        flex-direction: column;
        .movie-img {
          position: relative;
          overflow: hidden;
          &:hover {
            .overview {
              transform: translateY(0);
            }
          }
          img {
            display: block;
            width: 100%;
            height: 100%;
          }
          .review {
            position: absolute;
            top: 0;
            left: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            width: 40px;
            height: 40px;
            background-color: #c92502;
            color: #fff;
            border-radius: 0 0 16px 0;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1),
              0 2px 4px -1px rgba(0, 0, 0, 0.06);
          }
          .overview {
            line-height: 1.5;
            position: absolute;
            bottom: 0;
            background-color: rgba(201, 38, 2, 0.9);
            padding: 12px;
            color: #fff;
            transform: translateY(100%);
            transition: 0.3s ease-in-out all;
          }
        }
        .info {
          margin-top: auto;

          .title {
            margin-top: 8px;
            color: #fff;
            font-size: 20px;
          }
          .release {
            margin-top: 8px;
            color: #c9c9c9;
          }
          .button {
            margin-top: 8px;
          }
        }
      }
    }
  }
}
</style>
