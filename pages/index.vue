<template>
  <div class="container mx-auto my-10">

<!--    Toggle bonus/money price / Переключатель цен за бонусы/деньги-->
    <div class="toggle__wrapper sticky right-0 z-10 w-48 mx-auto sm:mr-0">
      <label for="toggle__button"
             class="flex items-center cursor-pointer bg-white px-4 py-2 rounded border border-gray-400 shadow-sm">

        <div class="relative">
          <input id="toggle__button" type="checkbox" class="hidden" v-model="payment"/>
          <div class="toggle__line w-10 h-4 bg-gray-400 rounded-full shadow-inner"></div>
          <div class="toggle__dot absolute w-6 h-6 bg-blue-500 rounded-full shadow inset-y-0 left-0"></div>
        </div>

        <div class="ml-3 text-gray-700 font-medium">
          {{ payment ? 'Деньги' : 'Бонусы' }}
        </div>
      </label>
    </div>

    <div class="wrapper mx-auto overflow-hidden relative">
      <h1 class="items__title mx-1 text-4xl text-center text-gray-800">Витрина</h1>

      <hr class="mb-2">

<!--      Filters / Фильтры-->
      <div class="items__filters flex flex-wrap">
        <base-filter :filter="filters.filterSubject" :subject-data="subjectData" @change="change"
                     class="">
          Все предметы
        </base-filter>

        <base-filter :filter="filters.filterGenre" :subject-data="genreData" @change="change"
                     class="">
          Все жанры
        </base-filter>

        <base-filter :filter="filters.filterGrade" :subject-data="gradeData" @change="change"
                     class="">
          Все классы
        </base-filter>

<!--        Search / Поиск-->
        <div class="mx-auto w-48 mt-2 sm:w-1/2 md:w-auto md:ml-0 lg:ml-auto lg:mr-0">
          <label for="searchId"></label>
          <input
            class="appearance-none w-full bg-gray-100 text-gray-700 border border-gray-400 rounded py-2 px-4 leading-tight focus:outline-none focus:bg-white focus:border-gray-500 shadow"
            type="text" v-model="search" id="searchId" placeholder="Поиск...">
        </div>
      </div>

      <hr class="mt-4 mb-2">

<!--      Content / Контент-->
      <div v-if="!loading" class="items__wrapper flex flex-wrap justify-center mx-0 sm:-mx-8 md:-mx-3 lg:-mx-5 xl:-mx-6">
        <div v-for="item in filteredData"
             :key="item.courseHash"
             class="mt-4 w-48 sm:w-1/3 sm:px-8 md:w-1/4 md:mt-16 md:px-3 lg:w-1/5 lg:px-5 xl:w-1/6 xl:px-6">
          <div class="item border text-gray-700 border-gray-500 rounded-lg shadow-md mx-auto overflow-hidden">
            <div class="image__wrapper">
              <img class="image" :src="'https://www.imumk.ru/svc/coursecover/'+ item.courseId" alt="">
            </div>
            <div class="item__description px-5 py-3">
              <div class="block truncate text-md" :title="item.title">{{ item.subject }}</div>
              <span class="block text-md -mt-2">{{ item.grade }} класс</span>
              <span class="block text-xs">{{ item.genre }}</span>
              <a class="text-blue-500 text-xs font-bold mt-4" :href="item.shopUrl">Подробнее</a>
              <div v-if="payment" class="text-white text-xs text-center font-bold bg-blue-500 rounded px-2">{{ item.price }}
                руб.
              </div>
              <div v-else class="text-white text-xs text-center font-bold bg-blue-500 rounded px-2">{{ item.priceBonus }}
                бонусов
              </div>
            </div>
          </div>
        </div>
      </div>
      <base-loader v-else />
    </div>
  </div>
</template>

<script>
import BaseLoader from "../components/BaseLoader";
import BaseFilter from "../components/BaseFilter";
import axios from "axios";
import _ from "lodash"

export default {
  async asyncData({$axios}) {
    const response = await $axios.post('https://krapipl.imumk.ru:8443/api/mobilev1/update')
      .then(response => {
        return response.data.items
      })

    return {
      obtainedData: response,
      subjectData: [...new Set(response.map(item => item.subject))],
      genreData: [...new Set(response.map(item => item.genre))],
      gradeData: [...new Set(response.map(item => item.grade))]
    }
  },

  components: {
    BaseLoader,
    BaseFilter
  },
  data() {
    return {
      filters: {
        filterSubject: {
          name: 'filterSubject',
          value: 'all',
        },
        filterGenre: {
          name: 'filterGenre',
          value: 'all',
        },
        filterGrade: {
          name: 'filterGrade',
          value: 'all',
        },
      },
      search: '',
      payment: true,
      loading: false
    }
  },

  methods: {
    change(payload, name) {
      this.filters[name].value = payload
    },

    someMethod(key, subject) {
      if (this.filters[key].value === 'all') {
        return this.obtainedData
      }
      return this.obtainedData.filter(t => t[subject] === this.filters[key].value)
    },
  },
  computed: {
    filteredData() {
      return _.intersection(
        this.someMethod('filterSubject', 'subject'),
        this.someMethod('filterGenre', 'genre'),
        this.someMethod('filterGrade', 'grade'),
        this.filteredList
      )
    },

    filteredList() {
      if (this.search) {
        return this.obtainedData.filter(post => post.title.toLowerCase().includes(this.search.toLowerCase()))
      }
      return this.obtainedData
    }
  }
}
</script>

<style>
.toggle__dot {
  top: -.25rem;
  left: -.25rem;
  transition: all 0.3s ease-in-out;
}

input:checked ~ .toggle__dot {
  transform: translateX(100%);
  background-color: #48bb78;
}

.toggle__wrapper {
  top: 20px;
}
</style>
