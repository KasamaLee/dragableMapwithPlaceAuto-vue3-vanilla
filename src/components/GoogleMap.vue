<template>
  <a-form class="">
    <div
      ref="mapElement"
      id="map"
      class="map-container"
      style="height: 50vh"
    ></div>

    <input type="text" id="pac-input" class="search-input" ref="inputElement" />
  </a-form>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { Loader } from "@googlemaps/js-api-loader";

const lat = ref<number>();
const long = ref<number>();

const inputElement = ref<HTMLInputElement | null>(null);
const mapElement = ref<HTMLElement | null>(null);
let map: google.maps.Map;
let marker: google.maps.Marker;

const key = ""; //your key

const defaultCenter = { lat: 15.1307268, lng: 100.4367551 };

onMounted(() => {
  const loader = new Loader({
    apiKey: key,
    version: "weekly",
    libraries: ["places"],
  });

  loader.load().then(async () => {
    const { Map } = (await google.maps.importLibrary(
      "maps"
    )) as google.maps.MapsLibrary;
    map = new Map(document.getElementById("map") as HTMLElement, {
      center: defaultCenter,
      zoom: 14,
      streetViewControl: false,
      fullscreenControl: false,
    });
    initMap();
    initAutocomplete();

    navigator.geolocation.getCurrentPosition(successCallback, errorCallback);
  });
});

const successCallback = (position: any) => {
  setLatLng(position.coords.latitude, position.coords.longitude);
};

const errorCallback = (error: any) => {
  console.log(error.message);
};

// # Initialize the map
async function initMap() {
  marker = new google.maps.Marker({
    position: map.getCenter(),
    map: map,
  });
  // # Function to update marker position
  const updateMarkerPosition = () => {
    const newCenter = map.getCenter();
    if (newCenter) {
      console.log("center change");
      marker.setPosition(newCenter);
      // inputElement.value = null;
    }
  };
  // Listen for map center changes
  map.addListener("center_changed", updateMarkerPosition);
  // Also listen for dragend to cover cases where the map is dragged
  map.addListener("dragend", () => {
    updateMarkerPosition;
    console.log("object");
    (inputElement.value as HTMLInputElement).value = "";
  });
}

function initAutocomplete() {
  // Create the search box and link it to the UI element.
  const input = document.getElementById("pac-input");
  const searchBox = new google.maps.places.SearchBox(input);

  map.controls[google.maps.ControlPosition.TOP_LEFT].push(input);
  // Bias the SearchBox results towards current map's viewport.
  map.addListener("bounds_changed", () => {
    searchBox.setBounds(map.getBounds());
  });

  // Listen for the event fired when the user selects a prediction and retrieve
  // more details for that place.
  searchBox.addListener("places_changed", () => {
    const places = searchBox.getPlaces();

    if (places.length == 0) {
      return;
    }

    const lat = places[0].geometry.location.lat();
    const lng = places[0].geometry.location.lng();
    setLatLng(lat, lng);
  });
}

const setLatLng = (latitude: number, longitude: number) => {
  lat.value = latitude;
  long.value = longitude;
  const latLng = new google.maps.LatLng(latitude, longitude);
  marker.setPosition(latLng);
  map.setCenter(latLng);
};
</script>

<style lang="scss" scoped>

.google-map {
  width: 100%;
  height: 270px;
}

.search-input {
  padding: 4px 11px;
  margin-top: 16px;
  width: 50%;
  font-size: 14px;
  border: 1px solid #d9d9d9;
  border-radius: 4px;
}px
.search-input:focus {
  border-color: gray;
  box-shadow: 0 0 0 2px rgba(5, 145, 255, 0.1);
  outline: 0;
}
</style>
