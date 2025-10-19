<template>
  <div class="px-8 lg:px-40 py-8" :class="colorMode.value === 'dark' ? 'bg-gray-700' : 'bg-gray-100'">
    <div class="flex justify-between sm:items-center items-start flex-col sm:flex-row gap-4">
      <SearchInput v-model="searchValue" class="order-1"/>
      <SelectFilter v-model="selectedValue" class="order-2"/>
    </div>
    <div class="grid grid-cols-1 lg:grid-cols-4 gap-8 sm:gap-20 pt-8 sm:pt-10">
      <div
        v-for="country in filteredCountries"
        :key="country.code"
      >
        <CountryCard :country="country" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

interface Country {
  name: string
  capital: string
  continent: string
  flagPng: string
  flagSvg: string
}

const selectedValue = ref('');
const searchValue = ref('');
const colorMode = useColorMode();

const { data: allCountries, error } = await useAsyncData('allCountries', async () => {
  try {
    const res = await $fetch('https://restcountries.com/v3.1/all?fields=name,flags,continents,capital,population')
    return res.map((c: any) => ({
      name: c.name.common,
      capital: c.capital[0] ?? 'N/A',
      continent: c.continents[0] ?? 'N/A',
      flagPng: c.flags.png,
      flagSvg: c.flags.svg,
      population: c.population
    }))
  } catch(e) {
    console.log(e)
  }
})

const continentMap: Record<string, string[]> = {
  europe: ['Europe'],
  asia: ['Asia'],
  africa: ['Africa'],
  oceania: ['Oceania'],
  americas: ['North America', 'South America'],
}

const filteredCountries = computed(() => {
  let list = allCountries.value ?? []

  // Filter by search term (country name)
  if (searchValue.value) {
    const searchTerm = searchValue.value.toLowerCase()
    list = list.filter(country =>
      country.name.toLowerCase().includes(searchTerm)
    )
  }

  // Filter by continent/region
  if (selectedValue.value && selectedValue.value !== 'all') {
    const continents = continentMap[selectedValue.value.toLowerCase()]
    if (continents) {
      list = list.filter(c => continents.includes(c.continent))
    }
  }

  return list
})

</script>

