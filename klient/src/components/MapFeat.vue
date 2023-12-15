<template>
    <div class="w-full md:top-[10px] md:left-[60px] md:w-auto z-[2] flex absolute px-5 py-7 gap-3 md:px-0 md:py-0 bg-transparent">
        <div class="relative flex-1 md:min-w-[400px]">
            <input type="text" class="py-4 pr-4 pl-8 text-md focus:outline-none shadow-md w-full shadow-md rounded-3xl" placeholder="Szukaj" v-model="searchQuery" @input="search" @focus="$emit('togSearchRes')">
                <div class="absolute top-0 right-[20px] items-center h-full flex">
                    <i class="fa-solid fa-magnifying-glass fa-flip-horizontal fa-lg" style="color: #000000;"></i>
                </div>
                <div class="absolute mt-2 w-full">
                    <div v-if="searchQuery && searchResult" class="overflow-y-scroll h-[300px] rounded-3xl bg-white">
                        <LoadingAnim v-if="!searchData"/>
                        <div v-else>
                          <div class="px-5 py-5 flex gap-x-5 hover:bg-zinc-400 hover:text-white cursor-pointer"
                                v-for="(result, index) in searchData"
                                :key="index"
                                @click="selectResult(result)"
                        >
                                <i class="fa-solid fa-location-dot fa-l" style="color: #0ae2e6;"></i>
                                <p class="text-sm">{{ result.place_name }}</p>
                            </div>  
                        </div>
                        
                    </div>
                    <div v-if="selResult" class="py-4 px-4 rounded-3xl bg-white mt-3">
                        <i @click="removeResult" class="fa-solid fa-xmark" style="color: #000000;"></i>
                        <h2 class="text-md">{{ selResult.text }}</h2>
                        <p class="text-xs mb-1">{{ selResult.city }}, {{ selResult.state }}</p>
                        <p class="text-xs">{{ selResult.properties.category }}</p>
                    </div>
                </div>
        </div>
        <div class="bg-white flex px-3 items-center shadow-md min-h-[50px] rounded-full" :class="{'bg-zinc-600': coords}" @click="$emit('getGeolocation')">
            <i class="fa-solid fa-location-crosshairs fa-xl" :class="{'fa-fade text-white': coords, 'animate-ping': fetchCoords}" ></i>
        </div>
    </div>
</template>

<script>
import { ref } from 'vue';
import axios from 'axios';
import LoadingAnim from './LoadingAnim.vue';
export default {
    props:["coords", "fetchCoords", "searchResult"],
    components: {LoadingAnim},
    setup(props, {emit}){
        const selResult = ref(null);
        const searchQuery = ref(null);
        const searchData = ref(null);
        const queryTimeout = ref(null);
        const search = () => {
            clearTimeout(queryTimeout.value);
            searchData.value = null;
            queryTimeout.value = setTimeout(async()=>{
                if(searchQuery.value !== ''){
                    const params = new URLSearchParams({
                        fuzzymatch: true,
                        language: "pl",
                        limit: 10,
                        proximity: props.coords ? `${props.coords.lng}, ${props.coords.lat}` : "0,0",
                    });
                    const getData = await axios.get(`http://localhost:3000/api/search/${searchQuery.value}?${params}`);
                    searchData.value = getData.data.features;
                    console.log(searchData.value);
                }
            },750)
        };
        const selectResult = (result) => {
            selResult.value = result;
            emit("plotResult", result.geometry);
        };
        const removeResult = () => {
            selResult.value = null;
            emit("removeResult");
        };
        return{searchData, searchQuery, queryTimeout, search, selectResult, selResult, removeResult};
    },
};
</script>