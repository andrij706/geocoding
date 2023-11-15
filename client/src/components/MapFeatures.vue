<script setup>
import {defineProps, ref, defineEmits} from 'vue'
import axios from 'axios'
import LoadingSpinner from './LoadingSinner.vue'

const emit = defineEmits()

const {coords, fetchCoords, searchResults} = defineProps(['coords', 'fetchCoords', 'searchResults'])

const searchQuery = ref(null)
const searchData = ref(null)
const queryTimeout = ref(null)
const selectedResults = ref(null)

const search = () => {
    clearTimeout(queryTimeout.value)

    searchData.value = null
    queryTimeout.value = setTimeout( async () => {
        if(searchQuery.value !== ''){
            const params = new URLSearchParams({
                fuzzyMatch: true,
                language: 'en',
                limit: 10,
                proximity: coords ? `${coords.lng},${coords.lat}` : '0,0'
            })
            const getData = await axios.get(`api/search/${searchQuery.value}?${params}`) // Localhost: (`http://localhost:3000/api/search/${searchQuery.value}?${params}`)
            searchData.value = getData.data.features
        }
    }, 750)
}

const selectResult = (result) => {
    selectedResults.value = result
    emit('plotResult', result.geometry)
}

const removeResult = () => {
    selectedResults.value = null
    emit('removeResult')
}

</script>

<template>
    <div>
        <div class="w-full md:w-auto absolute md:top-[40px] md:left-[60px] z-[2] flex gap-4 px-6 py-8 md:px-0 md:py-0 bg-transparent">
            <div class="relative flex-1 md:min-w-[350px]">
                <input 
                    class="pl-9 pr-4 py-3 text-sm focus:outline-none w-full shadow-md rounded-md"
                    type="text"
                    placeholder="Start your search"
                    v-model="searchQuery"
                    @input="search"
                    @focus="$emit('toggleSearchResults')"
                />
                <div class="absolute top-0 left-[8px] h-full flex items-center">
                    <i class="fa-solid fa-magnifying-glass-location"></i>
                </div>
                <div class="absolute mt-2 w-full">
                    <div v-if="searchQuery && searchResults" class="h-[200px] overflow-scroll bg-white rounded-md">
                        <LoadingSpinner v-if="!searchData"/>
                        <div v-else>
                            <div 
                                class="px-4 py-2 flex gap-x-2 cursor-pointer hover:bg-slate-600 hover:text-white"
                                v-for="(result, index) in searchData"
                                :key="index"
                                @click="selectResult(result)"
                            >
                                <i class="fas fa-map-marker-alt"></i>
                                <p class="text-xs">{{ result.place_name_en }}</p>
                            </div>
                        </div>
                    </div>
                    <div v-if="selectedResults" class="mt-2 px-4 py-3 bg-white rounded-md">
                        <i @click="removeResult" class="fa-regular fa-circle-xmark flex justify-end"></i>
                        <h1 class="text-lg">{{ selectedResults.text }}</h1>
                        <p class="text-xs mb-1">{{ selectedResults.properties.address }}, {{ selectedResults.city }}, {{ selectedResults.state }}</p>
                        <p class="text-sm">{{ selectedResults.properties.category }}</p>
                    </div>
                </div>
            </div>
            <div 
                class="px-4 bg-white flex items-center shadow-md rounded-md min-h-[45px]"
                :class="{'bg-slate-600': coords}"
                @click="$emit('getGeolocation')"    
            >
                <i 
                    class="fa-solid fa-location-arrow text-state-600 text-[18px]"
                    :class="{'text-white': coords, 'animate-pulse': fetchCoords}"
                    ></i>
            </div>
        </div>
    </div>
</template>