<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div
      class="z-20 flex justify-center relative bg-gray-900 bg-cover px-4 pt-8 pb-32"
    >
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input
            v-model="queryIp"
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            type="text"
            placeholder="Search for any IP address or leave empty to get your IP address"
          />
          <i
            class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fas fa-chevron-right"
            @click="getIpInfo"
          ></i>
        </div>
      </div>

      <!-- IP Info -->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>

    <!-- Leaflet Map -->
    <div class="h-full z-10" id="mapid"></div>
  </div>
</template>

<script>
import IPInfo from "@/components/IPInfo.vue";
import leaflet from "leaflet";
import axios from "axios";
import { onMounted, ref } from "vue";

export default {
  name: "HomeView",
  components: {
    IPInfo,
  },
  setup() {
    let mymap;
    const queryIp = ref("");
    const ipInfo = ref(null);
    const geoApiKey = ref("");

    onMounted(() => {
      mymap = leaflet.map("mapid").setView([-30.559482, 22.937506], 5);

      leaflet
        .tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
          maxZoom: 19,
          attribution: "© OpenStreetMap",
        })
        .addTo(mymap);
    });

    const getIpInfo = async () => {
      try {
        geoApiKey.value = process.env.VUE_APP_GEO_API_KEY;

        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country,city?apiKey=${geoApiKey.value}&ipAddress=${queryIp.value}`
        );

        const result = data.data;

        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };

        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (error) {
        alert(error.message);
      }
    };

    return { queryIp, ipInfo, getIpInfo };
  },
};
</script>
