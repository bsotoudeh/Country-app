<template>
    <NuxtLink :to="`/countryDetail?name=${encodeURIComponent(country.name)}`">
        <UCard
           :ui="{ body: '!p-0' }"
           class="hover:shadow-lg transition-shadow duration-200 h-[350px]"
        >
            <img
                :src="country.flagSvg || country.flagPng"
                :alt="`${country.name} flag`"
                loading="lazy"
                class="w-full aspect-[4/3] h-44 object-cover"
            />
            <div class="p-4">
                <h2 class="text-lg font-bold pb-2">{{ country.name }}</h2>
                <div class="text-sm font-semibold">Population: <span class="text-gray-500">{{ formatNumber(country.population) }}</span></div>
                <div class="text-sm font-semibold">Region: <span class="text-gray-500">{{ country.continent }}</span></div>
                <div class="text-sm font-semibold">Capital: <span class="text-gray-500">{{ country.capital }}</span></div>
            </div>
        </UCard>
    </NuxtLink>
</template>

<script setup lang="ts">
interface Props {
  country: {
    name: string
    capital: string
    continent: string
    flagPng: string
    flagSvg: string
    population: number
  }
}

defineProps<Props>()

const formatNumber = (num: number) => {
  return new Intl.NumberFormat().format(num)
}
</script>
