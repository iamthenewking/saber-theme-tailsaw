<template>
  <div class="flex flex-1 justify-end items-center text-right px-4">
    <div
      class="absolute md:relative w-full justify-end bg-white left-0 top-0 z-10 mt-7 md:mt-0 px-4 md:px-0"
      :class="{ 'hidden md:flex': !searching }"
    >
      <label for="search" class="hidden">Search</label>

      <input
        class="transition-fast relative block h-10 w-full lg:w-1/2 lg:focus:w-3/4 bg-gray-100 border border-gray-500 focus:border-blue-400 outline-none cursor-pointer text-gray-700 px-4 pb-0 pt-px"
        :class="{ 'transition-border': query, 'lg:w-3/4': query }"
        autocomplete="off"
        name="search"
        placeholder="Search"
        type="search"
        ref="search"
        @keyup="query = $refs.search.value"
        @keyup.esc="reset()"
        @keyup.enter="goTo()"
        @blur="!results && reset()"
      />

      <button
        v-if="query || searching"
        class="absolute top-0 right-0 leading-snug font-400 text-3xl text-blue-500 hover:text-blue-600 focus:outline-none pr-7 md:pr-3"
        @click="reset()"
      >
        &times;
      </button>

      <transition name="fade">
        <div
          v-if="query"
          class="absolute left-0 right-0 md:inset-auto w-full lg:w-3/4 text-left mb-4 md:mt-10"
        >
          <div
            class="flex flex-col bg-white border border-b-0 border-t-0 border-blue-400 rounded-b-lg shadow-lg mx-4 md:mx-0"
          >
            <a
              v-for="(result, index) in results"
              class="bg-white hover:bg-blue-100 border-b border-blue-400 text-xl cursor-pointer p-4"
              :class="{ 'rounded-b-lg': index === results.length - 1 }"
              :title="result.title"
              :key="result.permalink"
              @click.prevent="goTo(result.permalink)"
            >
              {{ result.title }}

              <span
                class="block font-normal text-gray-700 text-sm my-1"
                v-html="result.excerpt"
              />
            </a>

            <div
              v-if="!results.length"
              class="bg-white w-full hover:bg-blue-100 border-b border-blue-400 rounded-b-lg shadow cursor-pointer p-4"
            >
              <p class="my-0">
                No results for <strong>{{ query }}</strong>
              </p>
            </div>
          </div>
        </div>
      </transition>
    </div>

    <button
      title="Start searching"
      type="button"
      class="flex md:hidden bg-gray-100 hover:bg-blue-100 justify-center items-center border border-gray-500 rounded-full focus:outline-none h-10 px-3"
      @click.prevent="showInput()"
    >
      <img
        src="../assets/img/magnifying-glass.svg"
        alt="search icon"
        class="h-4 w-4 max-w-none"
      />
    </button>
  </div>
</template>

<script>
import Fuse from 'fuse.js'
import debounce from 'lodash.debounce'

export default {
  data() {
    return {
      searching: false,
      query: '',
      results: []
    }
  },
  methods: {
    showInput() {
      this.searching = true
      this.$nextTick(() => {
        this.$refs.search.focus()
      })
    },
    reset() {
      this.$refs.search.value = ''
      this.searching = false
      this.results = []
      this.query = ''
      this.$refs.search.blur()
    },
    goTo(to) {
      if (!to) to = this.results[0] && this.results[0].permalink
      if (!to) return
      this.$router.push(to)
      this.reset()
    },
    search() {
      if (window.__FUSE__) this.results = window.__FUSE__.search(this.query)
    }
  },
  watch: {
    query: debounce(
      function() {
        this.search()
      },
      50,
      {
        leading: true,
        trailing: true
      }
    )
  },
  async mounted() {
    if (!window.__FUSE__) {
      const database = await this.$fetchSearchDatabase()
      window.__FUSE__ = new Fuse(database, {
        shouldSort: true,
        keys: Object.keys(database[0])
      })
    }
  }
}
</script>

<style scoped>
input[name='search'] {
  background-image: url('../assets/img/magnifying-glass.svg');
  background-position: 0.8em;
  background-repeat: no-repeat;
  border-radius: 25px;
  text-indent: 1.2em;
}

input[name='search'].transition-border {
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
  border-top-left-radius: 0.5rem;
  border-top-right-radius: 0.5rem;
}

input[type='search']::-webkit-search-decoration,
input[type='search']::-webkit-search-cancel-button,
input[type='search']::-webkit-search-results-button,
input[type='search']::-webkit-search-results-decoration {
  -webkit-appearance: none;
}

.fade-enter-active {
  transition: opacity 0.5s;
}
.fade-leave-active {
  transition: opacity 0s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>