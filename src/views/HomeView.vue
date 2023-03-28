<template>
    <main class="container text-white">
        <div class="pt-4 mb-8 relative">
            <input
                type="text"
                v-model="searchQuery"
                @input="getSearchResults"
                placeholder="Search for a city or state"
                class="py-2 px-2 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus: shadow-[0px_1px_0_0_#004E71]"
            />
            <ul
                v-if="mapboxSearchResults"
                class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
            >
                <li
                    v-for="searchResult in mapboxSearchResults"
                    :key="searchResult.id"
                    class="py-2 cursor-pointer"
                    @click="PreviewCity(searchResult)"
                >
                    {{ searchResult.place_name }}
                </li>
            </ul>
        </div>
        <div class="flex flex-col gap-4">
            <Suspense>
                <CityList />
                <template #fallback>
                    <p>Loading...
                    </p>
                </template>
            </Suspense>
        </div>
    </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import SiteNavigation from "../components/SiteNavigation.vue";
import CityList from "../components/CityList.vue";

//send de data to route made in index.js
const router = useRouter();
const PreviewCity = (searchResult) => {
    console.log(searchResult);
    const [city, state] = searchResult.place_name.split(",");
    router.push({
        name: "cityView",
        params: { state: state.replaceAll(" ", ""), city: city },
        query: {
            lat: searchResult.geometry.coordinates[1],
            lng: searchResult.geometry.coordinates[0],
            preview: true
        },
    });
};

const mapboxAPIKey =
    "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);

const getSearchResults = () => {
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout(async () => {
        if (searchQuery.value !== "") {
            const result = await axios.get(
                `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
            );
            mapboxSearchResults.value = result.data.features;
            //console.log(mapboxSearchResults.value);
            return;
        }
        mapboxSearchResults.value = null;
    }, 300);
};
</script>
