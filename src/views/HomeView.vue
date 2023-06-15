<template>
  <main class="container text-white ">
    <div class="pt-4 mb-8 relative">
      <input type="text" v-model="searchQuery" placeholder="Search for a city or state" @input="getSearchResults"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
    </div>
    <!-- <ul class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]" v-if="mapboxSearchResults">
      <p v-if="searchError">
        Sorry, something went wrong, please try again.
      </p>
      <p v-if="!serverError && mapboxSearchResults.length === 0">
        No results match your query, try a different term.
      </p>
      <template v-else>
        <li v-for="getSearchResults in mapboxSearchResults" :key="searchResult.id" class="py-2 cursor-pointer"
          @click="previewCity(searchResult)">
          {{ searchResult.place_name }}
        </li>
      </template>
    </ul> -->
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList>
          <template #fallback>
            <CityCardSkeleton />
          </template>
        </CityList>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router';
import CityList from '../components/cityList.vue';
import CityCardSkeleton from '../components/CityCardSkeleton.vue';

const router = useRouter()
const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(',')
  router.push({
    name: 'cityView',
    params: { state: state.replaceAll(" ", ""), city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true
    }
  })
}

const mapboxAPIkey = `pk.eyJ1Ijoiam9obmtvbWFybmlja2kiiLCJhIjoiY2t5NjFz0DZvMHJkaDJJ1bWx60GvieGxreSJ9.IpojdT3U3NENknF6_WhR2Q`
const searchQuery = ref("")
const queryTimeout = ref(null)
const mapboxSearchResults = ref(null)
const searchError = ref(null)

const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?acccess_token=${mapboxAPIkey}`)
        mapboxSearchResults.value = result.data.features
      } catch (error) {
        searchError.value = true
      }
      return
    }
    mapboxSearchResults.value = null
  }, 3000)
}
</script>
