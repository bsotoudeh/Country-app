<template>
  <div class="min-h-screen" :class="colorMode.value === 'dark' ? 'bg-gray-800' : 'bg-gray-50'">
    <div class="container mx-auto 2xl:px-0 px-8 py-8">
      <!-- Back Button -->
      <div class="mb-8">
        <NuxtLink
          to="/"
          class="inline-flex items-center gap-2 px-8 py-3 bg-white dark:bg-gray-700 rounded shadow-md hover:shadow-lg transition-shadow duration-200"
        >
          <UIcon name="i-lucide-arrow-left" class="w-5 h-5" />
          <span>Back</span>
        </NuxtLink>
      </div>

      <!-- Loading State -->
      <div v-if="pending" class="flex justify-center items-center min-h-96">
        <div class="text-center">
          <UIcon name="i-lucide-loader-2" class="w-8 h-8 animate-spin mx-auto mb-4" />
          <p>Loading country details...</p>
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="text-center py-16">
        <UIcon name="i-lucide-alert-circle" class="w-16 h-16 text-red-500 mx-auto mb-4" />
        <h2 class="text-2xl font-bold mb-2">Error Loading Country</h2>
        <p class="text-gray-600 dark:text-gray-400">{{ error.message }}</p>
      </div>

      <!-- Country Details -->
      <div v-else-if="country" class="grid grid-cols-1 lg:grid-cols-2 gap-8 sm:gap-38 items-center">
        <!-- Flag -->
        <div class="order-1">
          <img
            :src="country.flags?.png || country.flags?.svg"
            :alt="`${country.name?.common} flag`"
            loading="lazy"
            class="w-full sm:h-96 h-48 object-cover shadow-lg"
          />
        </div>

        <!-- Country Information -->
        <div class="order-2">
          <!-- Country Name -->
          <div>
            <h1 class="text-4xl font-bold mb-4">{{ country.name?.common }}</h1>

          </div>

          <!-- Basic Information -->
          <div class="flex flex-col sm:flex-row gap-16 sm:gap-28">
            <div>
              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Native Name</h3>
                <p class="text-gray-600 dark:text-gray-400">
                  {{ getNativeName(country.name?.nativeName) }}
                </p>
              </div>

              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Population</h3>
                <p class="text-gray-600 dark:text-gray-400">
                  {{ formatNumber(country.population) }}
                </p>
              </div>

              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Region</h3>
                <p class="text-gray-600 dark:text-gray-400">{{ country.region }}</p>
              </div>

              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Sub Region</h3>
                <p class="text-gray-600 dark:text-gray-400">{{ country.subregion || 'N/A' }}</p>
              </div>

              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Capital</h3>
                <p class="text-gray-600 dark:text-gray-400">
                  {{ country.capital?.join(', ') || 'N/A' }}
                </p>
              </div>
            </div>

            <div>
              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Top Level Domain</h3>
                <p class="text-gray-600 dark:text-gray-400">
                  {{ country.tld?.join(', ') || 'N/A' }}
                </p>
              </div>

              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Currencies</h3>
                <p class="text-gray-600 dark:text-gray-400">
                  {{ getCurrencies(country.currencies) }}
                </p>
              </div>

              <div class="flex items-center">
                <h3 class="font-semibold text-lg mr-2">Languages</h3>
                <p class="text-gray-600 dark:text-gray-400">
                  {{ getLanguages(country.languages) }}
                </p>
              </div>
            </div>
          </div>

          <!-- Border Countries -->
          <div v-if="borderCountries && borderCountries.length > 0">
            <div class="flex flex-col sm:flex-row flex-wrap items-start sm:items-center gap-2 mt-8">
              <h3 class="font-semibold text-lg pr-4">Border Countries:</h3>
              <div class="flex flex-wrap gap-2">
                <NuxtLink
                  v-for="borderCountry in borderCountries"
                  :key="borderCountry.name?.common"
                  :to="`/countryDetail?name=${encodeURIComponent(borderCountry.name?.common)}`"
                  class="px-4 py-2 bg-white dark:bg-gray-700 shadow-sm hover:shadow-lg transition-shadow duration-200 text-sm"
                >
                  {{ borderCountry.name?.common }}
                </NuxtLink>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const route = useRoute()
const colorMode = useColorMode()
const error = useError()

// Get country name from query parameter - make it reactive
const countryName = computed(() => {
  const name = route.query.name as string
  if (!name) {
    error.value = {
      statusCode: 400,
      statusMessage: 'Country name is required'
    }
  }
  return name
})

// Fetch country data - make it reactive to route changes
const { data: country, pending, refresh: refreshCountry } = await useAsyncData(
  () => `country-${countryName.value}`,
  async () => {
    try {
      const response = await $fetch(`https://restcountries.com/v3.1/name/${encodeURIComponent(countryName.value)}`)
      return (response as any[])[0] // Get the first result
    } catch (err) {
      console.log(err)
    }
  },
  {
    watch: [countryName] // Watch for changes in countryName
  }
)

// Fetch border countries if they exist - make it reactive
const { data: borderCountries, refresh: refreshBorders } = await useAsyncData(
  () => `borders-${countryName.value}`,
  async () => {
    // Wait for country data to be available
    if (!country.value?.borders || country.value.borders.length === 0) {
      return []
    }
    try {
      const response = await $fetch(`https://restcountries.com/v3.1/alpha?codes=${country.value.borders.join(',')}`)
      return response as any[]
    } catch (err) {
      return []
    }
  },
  {
    default: () => [],
    watch: [countryName, country], // Watch for changes in both countryName and country data
    server: false // Ensure client-side refetching
  }
)

// Watch for route changes and refresh data
watch(() => route.query.name, async (newName, oldName) => {
  if (newName !== oldName && newName) {
    // First refresh the country data
    await refreshCountry()

    // Wait for the country data to be available, then refresh borders
    await nextTick()
    await refreshBorders()
  }
})

// Helper functions
const getNativeName = (nativeName: any) => {
  if (!nativeName) return 'N/A'
  const keys = Object.keys(nativeName)
  if (keys.length === 0) return 'N/A'
  const firstKey = keys[0]
  return nativeName[firstKey]?.common || 'N/A'
}

const getCurrencies = (currencies: any) => {
  if (!currencies) return 'N/A'
  return Object.values(currencies).map((curr: any) => `${curr.name} (${curr.symbol})`).join(', ')
}

const getLanguages = (languages: any) => {
  if (!languages) return 'N/A'
  return Object.values(languages).join(', ')
}

const formatNumber = (num: number) => {
  return new Intl.NumberFormat().format(num)
}

// Set page title
useHead({
  title: country.value ? `${country.value.name?.common} - Country Details` : 'Country Details'
})
</script>