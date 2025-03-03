<script setup>
import { ref } from "vue";

const ip = ref("");
const login = ref("");
const password = ref("");
const showLoginModal = ref(false);
const metrics = ref({
  status: "Не подключено",
  tcpPorts: "Нет данных",
  cpuLoad: "Нет данных",
  diskSpace: "Нет данных",
});

// Возможные значения для статуса
const statusOptions = ["Не подключено", "✅ В сети", "❌ Ошибка"];

// Возможные значения для TCP-портов
const tcpPortsOptions = ["Нет данных", "Открыты", "Закрыты"];

// Возможные значения для загрузки процессора
const cpuLoadOptions = ["Нет данных", "🔥 10%", "🔥 30%", "🔥 50%", "🔥 70%", "🔥 90%", "🔥 100%"];

// Возможные значения для дискового пространства
const diskSpaceOptions = ["Нет данных", "100 GB", "200 GB", "500 GB", "1 TB", "2 TB"];

// Функция для получения случайного значения из массива
const getRandomItem = (array) => array[Math.floor(Math.random() * array.length)];

const startConnection = () => {
  const storedDevices = JSON.parse(localStorage.getItem("devices")) || [];
  const device = storedDevices.find((d) => d.ip === ip.value);

  if (!device) {
    metrics.value.status = "❌ Ошибка: IP-адрес не найден в системе.";
    return;
  }
  showLoginModal.value = true;
};

const connectToDevice = () => {
  showLoginModal.value = false;

  setTimeout(() => {
    // Устанавливаем случайные значения для метрик только один раз
    metrics.value.status = getRandomItem(statusOptions);
    metrics.value.tcpPorts = getRandomItem(tcpPortsOptions);
    metrics.value.cpuLoad = getRandomItem(cpuLoadOptions);
    metrics.value.diskSpace = getRandomItem(diskSpaceOptions);
  }, 1000);
};

// Функция для получения класса в зависимости от значения метрики
const getStatusClass = (status) => {
  if (status.includes("Ошибка") || status === "Не подключено") {
    return "text-red"; // Плохие новости — красный
  }
  if (status.includes("В сети") || status.includes("Открыты") || status.includes("🔥 10%")) {
    return "text-green"; // Хорошие новости — зеленый
  }
  return "text-gray"; // Нейтральные новости — серый
};

const getCpuLoadClass = (cpuLoad) => {
  if (cpuLoad.includes("🔥 90%") || cpuLoad.includes("🔥 100%")) {
    return "text-red"; // Плохая загрузка процессора — красный
  }
  if (cpuLoad.includes("🔥 10%") || cpuLoad.includes("🔥 30%")) {
    return "text-green"; // Низкая загрузка процессора — зеленый
  }
  return "text-gray"; // Нейтральное состояние — серый
};

const getDiskSpaceClass = (diskSpace) => {
  if (diskSpace === "Нет данных") {
    return "text-gray"; // Нет данных — серый
  }
  if (diskSpace === "100 GB" || diskSpace === "200 GB") {
    return "text-red"; // Малый объем — красный
  }
  return "text-green"; // Большой объем — зеленый
};

</script>

<template>
  <div class="connect">
    <div class="connect__header">
      <h2>Подключение к устройству</h2>
      <div class="connect__header__container">
        <input v-model="ip" placeholder="Введите IP-адрес" />
        <button @click="startConnection">Запуск сбора</button>
      </div>
    </div>

    <!-- Модальное окно логина -->
    <div v-if="showLoginModal" class="device__bck">
      <div class="device__bck__popup">
        <h3>Введите логин и пароль</h3>
        <input v-model="login" placeholder="Логин" />
        <input v-model="password" type="password" placeholder="Пароль" />
        <button @click="connectToDevice">Подключиться</button>
      </div>
    </div>

    <!-- Вывод метрик -->
    <div class="metrics-container">
      <div :class="getStatusClass(metrics.status)" class="metric"><b>Статус устройства:</b> {{ metrics.status }}</div>
      <div :class="getStatusClass(metrics.tcpPorts)" class="metric"><b>TCP-порты:</b> {{ metrics.tcpPorts }}</div>
      <div :class="getCpuLoadClass(metrics.cpuLoad)" class="metric"><b>Загрузка процессора:</b> {{ metrics.cpuLoad }}</div>
      <div :class="getDiskSpaceClass(metrics.diskSpace)" class="metric"><b>Свободное дисковое пространство:</b> {{ metrics.diskSpace }}</div>
    </div>
  </div>
</template>

<style>
.connect {
  text-align: center;
  font-family: Arial, sans-serif;
  width: 90%;
  margin: 10px auto 0 auto;
  border-top: 1px solid #0077ff;
}

.connect__header {
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: space-between;
}

.connect__header h2 {
  font-size: 2.094vw;
  font-weight: bold;
  color: #0077ff;
}

.connect__header__container {
  display: flex;
  align-items: center;
  gap: 0.524vw;
}

.connect__header__container input {
  border: 2px solid #0077ff;
  border-radius: 0.524vw;
  color: #0077ff;
  font-size: 1.361vw;
  width: 14.974vw;
  height: 2.199vw;
  padding-left: 0.524vw;
}

.connect__header__container input::placeholder {
  color: #0077ff9e;
}

.connect__header__container button {
  border: 2px solid #0077ff;
  border-radius: 0.524vw;
  color: #0077ff;
  font-size: 1.361vw;
  height: 2.199vw;
  background-color: transparent;
  outline: none;
  cursor: pointer;
}

.connect__header__container button:hover {
  background-color: #0077ff;
  color: white;
}


.metrics-container {
  display: flex;
  justify-content: center;
  gap: 1.047vw;
  margin-top: 2.094vw;
}

.metric {
  padding: 10px;
  border: 1px solid #0077ff;
  border-radius: 0.524vw;
  min-width: 15.707vw;
  text-align: center;
  font-size: 1.571vw;
  color: #0077ff;
}

.text-red {
  color: red;
}

.text-green {
  color: green;
}

.text-gray {
  color: gray;
}

.metric b{
  font-size: 1.571vw;
  color: #0077ff;
}
</style>
