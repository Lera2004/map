<template>
  <div class="flex flex-col items-center justify-center h-screen ">
    <div class="map-container">
      <div class="flex-container">
      <div class="map" ref="mapContainer"></div>
      <div class="legend">
        <div class="legend-item">
          <span class="legend-color" style="background-color: green;"></span>
          <span class="legend-text">Здорове</span>
        </div>
        <div class="legend-item">
          <span class="legend-color" style="background-color: red;"></span>
          <span class="legend-text">Хворе</span>
        </div>
        <div class="legend-item">
          <span class="legend-color" style="background-color: orange;"></span>
          <span class="legend-text">Пошкоджене</span>
        </div>
        <div class="legend-item">
          <span class="legend-color" style="background-color: blue;"></span>
          <span class="legend-text">Молоде</span>
        </div>
      </div>
      <label for="treeType" class="text-lg font-bold text-indigo-500 mt-4">Виберіть тип дерева:</label>
      <select v-model="selectedType" @change="filterMarkers"
        class="my-select mt-2 p-2 border border-gray-300 rounded w-full md:w-1/2">
        <option value="all">All</option>
        <option v-for="type in uniqueTypes" :value="type" :key="type">
          {{ type }}
        </option>
      </select>
      <label for="treeState" class="text-lg font-bold text-indigo-500 mt-4">Виберіть стан дерева:</label>
      <select v-model="selectedState" @change="filterMarkers"
        class="my-select mt-2 p-2 border border-gray-300 rounded w-full md:w-1/2">
        <option value="all">All</option>
        <option v-for="state in uniqueStates" :value="state" :key="state">
          {{ state }}
        </option>
      </select>
      </div>
    </div>
    <div v-if="isModalOpen" class="modal">
      <div class="modal-content flex">
        <div class="modal-image-container">
          <img v-if="selectedTree.PhotoLink" :src="getImageUrl(selectedTree.PhotoLink)" class="tree-image">
        </div>
        <div class="modal-text-container">
          <p class="modal-text"><strong>ID:</strong> {{ selectedTree._id }}</p>
          <p class="modal-text"><strong>Назва:</strong> {{ selectedTree.Tree_Name }}</p>
          <p class="modal-text"><strong>Стан:</strong> {{ selectedTree.Tree_State }}</p>
          <p class="modal-text"><strong>Локація:</strong> {{ selectedTree.Location }}</p>
          <button @click="isModalOpen = false" class="modal-close-btn">Закрити</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import L from "leaflet";
import axios from "axios";

const treeData = ref([]);
const map = ref();
const mapContainer = ref();
const selectedType = ref("all");
const selectedState = ref("all");
let markerLayer = L.layerGroup();
const uniqueTypes = ref([]);
const uniqueStates = ref([]);

const getImageUrl = (photoLink) => {
  const imageUrl = `http://10.0.0.104:3000/uploads/${photoLink}`;
  console.log('Image URL:', imageUrl);
  return imageUrl;
};

const updateMarkers = () => {
  markerLayer.clearLayers();
  const filteredData = treeData.value.filter(item =>
    (selectedType.value === "all" || item.Tree_Name === selectedType.value) &&
    (selectedState.value === "all" || item.Tree_State === selectedState.value)
  );

  filteredData.forEach((coordForMarker) => {
    let markerColor = "green";

    switch (coordForMarker.Tree_State) {
      case "Здорове":
        markerColor = "green";
        break;
      case "Хворе":
        markerColor = "red";
        break;
      case "Пошкоджене":
        markerColor = "orange";
        break;
      case "Молоде":
        markerColor = "blue";
        break;
      default:
        markerColor = "green";
    }

    const latitude = coordForMarker.Latitude;
    const longitude = coordForMarker.Longitude;

    if (latitude !== undefined && longitude !== undefined) {
      const markerInstance = L.circleMarker([latitude, longitude], {
        radius: 5,
        fillColor: markerColor,
        color: markerColor
      });

      markerInstance.on('click', () => {
        onMarkerClick(coordForMarker);
      });

      markerInstance.addTo(markerLayer);
    }
  });

  markerLayer.addTo(map.value);
};

const filterMarkers = () => {
  updateMarkers();
};

const onMarkerClick = (marker) => {
  selectedTree.value = marker;
  console.log('Marker clicked:', marker);
  isModalOpen.value = true;
};

onMounted(() => {
  axios.get("http://10.0.0.104:3000/trees").then((res) => {
    treeData.value = res.data;
    uniqueTypes.value = [...new Set(treeData.value.map(item => item.Tree_Name))];
    uniqueStates.value = [...new Set(treeData.value.map(item => item.Tree_State))];
    updateMarkers();
  })

  map.value = L.map(mapContainer.value).setView([47.8388, 35.1396], 13);
  L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
  }).addTo(map.value);
});

const isModalOpen = ref(false);
const selectedTree = ref(null);
</script>

<style scoped>
.map {
  width: 100%;
  height: 500px;
}

.legend {
  margin-left: 16px;
  margin-top: 16px;
  margin-bottom: 16px;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
}

.legend-color {
  width: 20px;
  height: 20px;
  margin-right: 8px;
}

.my-select {
  width: 100%;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  width: 90%;
  display: flex;
  justify-content: space-between;
  text-align: left;
}

.modal-image-container {
  margin-right: 20px;
}

.tree-image {
  max-width: 100%;
}

.modal-text-container {
  flex: 2;
}

.modal-close-btn {
  background-color: red;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 10px;
  align-self: center;
}

@media (min-width: 375px) {
  .map-container {
    margin: 20px;
  }

  .legend {
    margin-right: 20px;
  }

  .my-select {
    margin-right: 20px;
  }
  .modal-content {
    width: 93%;
  }
}

@media (min-width: 640px) {
  .map-container {
    width: 90%;
  }

  .legend {
    margin-right: 16px;
  }

  .my-select {
    margin-right: 16px;
  }

  .modal-content {
    width: 80%;
  }
}

@media (min-width: 768px) {
  .map-container {
    width: 90%;
  }

  .legend {
    margin-right: 16px;
  }

  .my-select {
    margin-right: 16px;
  }

  .modal-content {
    width: 70%;
  }
}

@media (min-width: 1024px) {
  .map-container {
    width: 90%;
  }

  .legend {
    margin-right: 16px;
  }

  .my-select {
    margin-right: 16px;
  }

  .modal-content {
    width: 60%;
  }
}
</style>