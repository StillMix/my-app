<script setup>
import { ref, computed, onMounted, watch } from "vue";

const STORAGE_KEY = "devices";
const getStoredDevices = () => JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
const saveDevices = (devices) => localStorage.setItem(STORAGE_KEY, JSON.stringify(devices));

const deviceList = ref(getStoredDevices());
const searchQuery = ref("");
const selectedDevice = ref(null);
const showModal = ref(false);
const showDeleteConfirm = ref(false);
const showEditModal = ref(false);
const showAddModal = ref(false);
const newDevice = ref({ name: "", ip: "", type: "" });

watch(deviceList, (newList) => saveDevices(newList), { deep: true });

const filteredDevices = computed(() =>
  deviceList.value.filter((d) =>
    d.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
);

const openDetails = (device) => {
  selectedDevice.value = device;
  showModal.value = true;
  document.body.style.overflow = "hidden";
};

const openModal = () => {
  document.body.style.overflow = "hidden";
};

const addDevice = () => {
  if (!newDevice.value.name || !newDevice.value.ip) return;
  deviceList.value.push({ id: Date.now(), ...newDevice.value });
  newDevice.value = { name: "", ip: "", type: "" };
  showAddModal.value = false;
  document.body.style.overflow = "";
};

const editDevice = (device) => {
  selectedDevice.value = { ...device };
  document.body.style.overflow = "hidden";
  showEditModal.value = true;
};

const saveEdit = () => {
  if (!selectedDevice.value) return;

  const index = deviceList.value.findIndex(
    (d) => d.id === selectedDevice.value.id
  );

  if (index !== -1) {
    deviceList.value[index] = { ...selectedDevice.value };
  }

  showEditModal.value = false;
  document.body.style.overflow = "";
};

const deleteDevice = () => {
  deviceList.value = deviceList.value.filter(
    (d) => d.id !== selectedDevice.value.id
  );
  showDeleteConfirm.value = false;
  document.body.style.overflow = "";
};

const closeModal = () => {
  showModal.value = false;
  showEditModal.value = false;
  showAddModal.value = false;
  showDeleteConfirm.value = false;
  selectedDevice.value = null;
  document.body.style.overflow = "";
};
</script>


<template>
  <div>
    <div class="device__header">
      <h2>Список сетевых устройств</h2>
      <input
        v-model="searchQuery"
        placeholder="Введите устройство для поиска"
      />
    </div>

    <table class="device__table">
      <thead>
        <tr>
          <th>Название</th>
          <th>IP-адрес</th>
          <th>Тип</th>
          <th>Действия</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="device in filteredDevices" :key="device.id">
          <td @dblclick="openDetails(device)">{{ device.name }}</td>
          <td @dblclick="openDetails(device)">{{ device.ip }}</td>
          <td @dblclick="openDetails(device)">{{ device.type }}</td>
          <td>
            <button class="device__table__btn" @click="editDevice(device)">
              Редактировать
            </button>
            <button
              class="device__table__btn"
              @click="
                showDeleteConfirm = true;
                openModal();
                selectedDevice = device;
              "
            >
              Удалить
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <button
      class="device__btnShowAdd"
      @click="
        showAddModal = true;
        openModal();
      "
    >
      Добавить
    </button>

    <div v-if="showAddModal" class="device__bck">
      <div class="device__bck__popup">
        <h3>Добавить устройство</h3>
        <input v-model="newDevice.name" placeholder="Название" />
        <input v-model="newDevice.ip" placeholder="IP-адрес" />
        <input v-model="newDevice.type" placeholder="Тип" />
        <button @click="addDevice">Добавить</button>
        <button @click="closeModal">Закрыть</button>
      </div>
    </div>

    <!-- Модальное окно характеристик -->
    <div v-if="showModal" class="device__bck">
      <div class="device__bck__popup">
        <h3>Характеристики {{ selectedDevice.name }}</h3>
        <p><b>IP:</b> {{ selectedDevice.ip }}</p>
        <p><b>Тип:</b> {{ selectedDevice.type }}</p>
        <button @click="closeModal">Закрыть</button>
      </div>
    </div>

    <div v-if="showEditModal" class="device__bck">
      <div v-if="selectedDevice" class="device__bck__popup">
        <h3>Редактирование устройства</h3>

        <!-- Используйте v-model для двусторонней связи -->
        <input v-model="selectedDevice.name" placeholder="Название" />
        <input v-model="selectedDevice.ip" placeholder="IP" />
        <input v-model="selectedDevice.type" placeholder="Тип" />

        <button @click="saveEdit">Сохранить</button>
        <button @click="closeModal">Отмена</button>
      </div>
    </div>

    <!-- Диалог подтверждения удаления -->
    <div v-if="showDeleteConfirm" class="device__bck">
      <form class="device__bck__popup">
        <h3>Удалить {{ selectedDevice.name }}?</h3>
        <button @click="deleteDevice">Удалить</button>
        <button @click="closeModal">Отмена</button>
      </form>
    </div>
  </div>
</template>

<style>
.device__header {
  display: flex;
  width: 90%;
  justify-content: space-between;
  align-items: center;
  margin: 10px auto 0 auto;
}
.device__header h2 {
  font-size: 2.094vw;
  font-weight: bold;
  color: #0077ff;
}
.device__header input {
  border-radius: 40px;
  border-color: #0077ff;
  color: #0077ff;
  height: 3.094vw;
  width: 23.56vw;
  padding-left: 0.524vw;
  font-size: 1.361vw;
}

.device__header input::placeholder {
  color: #0077ff60;
  font-size: 1.361vw;
}

.device__table {
  width: 90%;
  margin: 5px auto 0 auto;
}
.device__table thead tr th {
  font-weight: bold;
  font-size: 1.466vw;
  text-align: left;
  color: #0077ff;
}

.device__table thead tr th:last-child {
  text-align: right;
}

.device__table tbody tr td {
  text-align: left;
  color: #0077ff;
  user-select: none;
  cursor: pointer;
  font-size: 1.466vw;
}

.device__table tbody tr td:hover {
  opacity: 0.8;
}

.device__table tbody tr td:last-child {
  text-align: right;
}

.device__table__btn {
  border-radius: 0.524vw;
  cursor: pointer;
  outline: none;
  background: transparent;
  color: #005ac1;
  border: 2px solid #005ac1;
  font-size: 1.466vw;
}

.device__table__btn:last-child {
  margin-left: 5px;
}
.device__table__btn:hover {
  opacity: 0.8;
}

.device__btnShowAdd {
  width: 90%;
  margin: 1px auto 0 auto;
  display: block;
  color: #005ac1;
  border: 2px solid #005ac1;
  font-size: 1.466vw;
  cursor: pointer;
  outline: none;
  background: transparent;
}
.device__btnShowAdd:hover {
  opacity: 0.8;
  background-color: #005ac1;
  color: white;
}
.device__bck {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 5;
  background: #00489a92;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: auto;
}
.device__bck__popup {
  width: 36.649vw;
  border-radius: 2.094vw;
  background-color: #0077ff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.524vw;
  padding-bottom: 3.141vw;
  padding-top: 1.047vw;
}

.device__bck__popup h3 {
  color: white;
  font-weight: bold;
  font-size: 2.094vw;
}

.device__bck__popup p {
  color: white;
  font-size: 1.571vw;
}

.device__bck__popup input {
  border: 2px solid white;
  outline: none;
  width: 90%;
  border-radius: 5px;
  background: transparent;
  color: white;
  font-size: 1.257vw;
}

.device__bck__popup input::placeholder {
  color: white;
  font-size: 1.257vw;
}

.device__bck__popup button {
  width: 90%;
  height: 3.141vw;
  background-color: white;
  color: #0077ff;
  border: none;
  outline: none;
  cursor: pointer;
  font-size: 1.257vw;
}
.device__bck__popup button:hover {
  background-color: #0077ff;
  color: white;
}
</style>
