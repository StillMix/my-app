<script setup>
import { ref, onMounted, watch } from "vue";

// Инициализируем переменную для устройств
const devices = ref([]);

// Загружаем данные устройств из localStorage или используем начальные данные
const loadDevicesFromStorage = () => {
  const savedDevices = localStorage.getItem("devices");
  if (savedDevices) {
    const parsedDevices = JSON.parse(savedDevices);
    // Добавляем координаты по умолчанию, если их нет
    return parsedDevices.map(device => ({
      ...device,
      latitude: device.latitude || 55.751244,  // Широта по умолчанию
      longitude: device.longitude || 37.618423,  // Долгота по умолчанию
    }));
  }
  return devices.value;
};

const selectedDevice = ref(null);
let mapInstance = null;
const markers = new Map(); // Хранит все маркеры

// Загружаем устройства из локального хранилища при монтировании компонента
onMounted(() => {
  // Инициализируем устройства
  const loadedDevices = loadDevicesFromStorage();
  devices.value = loadedDevices;
  const interval = setInterval(() => {
    const savedDevices = localStorage.getItem("devices");
    if (savedDevices) {
      const parsedDevices = JSON.parse(savedDevices);
      if (JSON.stringify(devices.value) !== JSON.stringify(parsedDevices)) {
        devices.value = parsedDevices;
      }
    }
  }, 1000);

  // Если данные из localStorage есть, выбираем первое устройство
  if (loadedDevices.length > 0) {
    selectedDevice.value = loadedDevices[0];
  }

  ymaps.ready(() => {
    mapInstance = new ymaps.Map("map", {
      center: [55.751244, 37.618423],
      zoom: 10,
    });

    // Добавляем все устройства на карту
    devices.value.forEach((device) => {
      addMarker(device);
    });
  });
});

// Функция добавления маркера
const addMarker = (device) => {
  const marker = new ymaps.Placemark([device.latitude, device.longitude], {
    balloonContent: device.name,
  });
  mapInstance.geoObjects.add(marker);
  markers.set(device.id, marker);
};

// Следим за изменением выбранного устройства и централизуем карту
watch(selectedDevice, (newDevice) => {
  if (newDevice && mapInstance) {
    mapInstance.setCenter([newDevice.latitude, newDevice.longitude], 12);
  }
});

// Следим за изменением координат и обновляем карту
watch(
  () => selectedDevice.value ? [selectedDevice.value.latitude, selectedDevice.value.longitude] : null,
  ([lat, lon]) => {
    if (selectedDevice.value && mapInstance) {
      const { id, name } = selectedDevice.value;

      // Обновляем устройство в данных
      const updatedDevice = { ...selectedDevice.value, latitude: lat, longitude: lon };
      
      // Сохраняем обновленные данные в localStorage
      const deviceIndex = devices.value.findIndex(device => device.id === updatedDevice.id);
      if (deviceIndex !== -1) {
        devices.value[deviceIndex] = updatedDevice;
        localStorage.setItem("devices", JSON.stringify(devices.value));
      }

      // Удаляем старый маркер
      if (markers.has(id)) {
        mapInstance.geoObjects.remove(markers.get(id));
      }

      // Добавляем новый маркер и центрируем карту
      addMarker({ id, latitude: lat, longitude: lon, name });
      mapInstance.setCenter([lat, lon], 12);
    }
  }
);

watch(selectedDevice, (newDevice) => {
  if (newDevice) {
    // Обновляем localStorage с новым устройством
    const deviceIndex = devices.value.findIndex(device => device.id === newDevice.id);
    if (deviceIndex !== -1) {
      devices.value[deviceIndex] = newDevice;
      localStorage.setItem("devices", JSON.stringify(devices.value));
    }
  }
}, { deep: true });
</script>

<template>
  <div class="MapY">
    <h2>Выбор и редактирование устройств</h2>

    <!-- Выпадающий список для выбора устройства -->
    <select v-model="selectedDevice">
      <option v-for="device in devices" :key="device.id" :value="device">
        {{ device.name }}
      </option>
    </select>

    <!-- Инпуты для редактирования широты и долготы -->
    <div v-if="selectedDevice">
      <input v-model="selectedDevice.latitude" type="number" step="0.0001" placeholder="Широта" />
      <input v-model="selectedDevice.longitude" type="number" step="0.0001" placeholder="Долгота" />
    </div>

    <!-- Карта -->
    <div id="map"></div>
  </div>
</template>

<style scoped>
#map {
  width: 100%;
  height: 400px;
}

.MapY {
  text-align: center;
  font-family: Arial, sans-serif;
  width: 90%;
  margin: 10px auto;
}

.MapY h2 {
  text-align: left;
  font-size: 2vw;
  font-weight: bold;
  color: #0077ff;
}

select, input {
  padding: 8px;
  font-size: 16px;
  margin: 5px;
}
</style>
