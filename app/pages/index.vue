<template>
  <div class="px-8 lg:px-32 py-8" :class="colorMode.value === 'dark' ? 'bg-gray-700' : 'bg-gray-100'">
    <div class="flex justify-between sm:items-center items-start flex-col sm:flex-row gap-4">
      <SearchInput v-model="searchValue" class="order-1"/>
      <SelectFilter v-model="selectedValue" class="order-2"/>
    </div>
    <div class="grid grid-cols-1 lg:grid-cols-4 gap-20 pt-10">
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

// const allCountries: Country[] = [
//   { name: 'France', code: 'FR', population: 67000000, region: 'Europe', capital: 'Paris' },
//   { name: 'Germany', code: 'DE', population: 83000000, region: 'Europe', capital: 'Berlin' },
//   { name: 'Russia', code: 'RU', population: 146000000, region: 'Europe', capital: 'Moscow' },
//   { name: 'Singapore', code: 'SG', population: 5900000, region: 'Asia', capital: 'Singapore' },
//   { name: 'Netherlands', code: 'NL', population: 17000000, region: 'Europe', capital: 'Amsterdam' },
//   { name: 'Iran', code: 'IR', population: 83000000, region: 'Asia', capital: 'Tehran' },
//   { name: 'United States', code: 'US', population: 331000000, region: 'Americas', capital: 'Washington, D.C.' },
//   { name: 'Canada', code: 'CA', population: 38000000, region: 'Americas', capital: 'Ottawa' },
//   { name: 'Australia', code: 'AU', population: 25000000, region: 'Oceania', capital: 'Canberra' },
//   { name: 'New Zealand', code: 'NZ', population: 5000000, region: 'Oceania', capital: 'Wellington' },
//   { name: 'South Africa', code: 'ZA', population: 60000000, region: 'Africa', capital: 'Pretoria' },
//   { name: 'India', code: 'IN', population: 1380000000, region: 'Asia', capital: 'New Delhi' },
//   { name: 'China', code: 'CN', population: 1400000000, region: 'Asia', capital: 'Beijing' },
// ]

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

