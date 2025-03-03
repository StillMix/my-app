<script setup>
import { ref, onMounted } from "vue";
import { Network } from "vis-network/standalone";

// Состояние для URL, узлов и связей
const url = ref("");
const searchQuery = ref("");  // Новый стейт для поиска по имени или ID
const nodes = ref([]);
const edges = ref([]);
const network = ref(null);

// Пример данных для графа
const exampleRoute = [
  { id: 1, label: "Узел 1", x: 0, y: 0 },
  { id: 2, label: "Узел 2", x: 150, y: 0 },
  { id: 3, label: "Узел 3", x: 150, y: 150 },
  { id: 4, label: "Узел 4", x: 0, y: 150 },
];

// Инициализация карты и графа после монтирования компонента
onMounted(() => {
  const container = document.getElementById("network"); // Получаем контейнер
  if (container) {
    const data = { nodes: nodes.value, edges: edges.value };
    const options = {
      nodes: {
        shape: "image",
        image: "https://avatars.mds.yandex.net/i?id=e5ec6feaa65293917a5a0451be04132e_l-10285207-images-thumbs&n=13",
        size: 10,
      },
      physics: {
        enabled: true,
        solver: "barnesHut",
        barnesHut: {
          gravitationalConstant: -20000,
          springLength: 100,
          springConstant: 0.04,
        },
        stabilization: { iterations: 50 },
      },
      interaction: {
        dragNodes: true,
        hover: true,
        zoomView: true, // Разрешаем зумирование
      },
      manipulation: {
        enabled: true, // Включаем режим манипуляции для добавления и редактирования
        addEdge: true, // Разрешаем добавление связей
        editNode: (data, callback) => {
          const newLabel = prompt("Введите новое имя для узла", data.label);
          if (newLabel !== null) {
            // Обновляем метку узла
            data.label = newLabel;
            callback(data);
          }
        },
      },
    };

    // Инициализация сети
    network.value = new Network(container, data, options);

    // Инициализация с примерными данными для визуализации
    updateGraph(exampleRoute);
  }
});

// Функция для обновления данных графа
const updateGraph = (route) => {
  // Добавляем узлы
  nodes.value = route.map((route) => ({
    id: route.id,
    label: route.label, // Начальная метка
    x: route.x,
    y: route.y,
  }));

  // Добавляем связи между узлами
  edges.value = [
    { from: 1, to: 2 },
    { from: 2, to: 3 },
    { from: 3, to: 4 },
  ];

  // Обновляем данные графа
  if (network.value) {
    network.value.setData({ nodes: nodes.value, edges: edges.value });
  }
};

// Функция для трассировки маршрута (имитация)
const traceRoute = async () => {
  // В реальной ситуации нужно использовать API или сервер для трассировки
  if (url.value) {
    // Пример использования URL, можно заменить на асинхронный запрос
    console.log("Запрашиваем маршрут для URL:", url.value);

    // Симулируем маршрут с заранее заданными узлами
    updateGraph(exampleRoute); // Используем тот же пример
  } else {
    alert("Введите URL для трассировки");
  }
};

// Функция для поиска и выделения узла по имени или ID
const highlightNode = () => {
  const query = searchQuery.value.trim().toLowerCase();

  // Если введено число, ищем по id
  if (query && !isNaN(query)) {
    const nodeId = parseInt(query);
    const node = nodes.value.find(n => n.id === nodeId);
    if (node) {
      // Выделяем узел по ID
      network.value.selectNodes([node.id]);
    } else {
      alert("Узел с таким ID не найден!");
    }
  }
  // Иначе ищем по метке (label)
  else if (query) {
    const node = nodes.value.find(n => n.label.toLowerCase() === query);
    if (node) {
      // Выделяем узел по label
      network.value.selectNodes([node.id]);
    } else {
      alert("Узел с таким именем не найден!");
    }
  } else {
    alert("Введите ID или имя узла!");
  }
};
</script>

<template>
  <div class="traceroute">
    <h2>Трассировка маршрута</h2>

    <!-- Поле для ввода URL -->
    <input v-model="url" type="text" placeholder="Введите URL" />

    <!-- Кнопка для запуска трассировки -->
    <button @click="traceRoute">Запустить трассировку</button>

    <!-- Поле для ввода имени или ID узла для поиска -->
    <input v-model="searchQuery" type="text" placeholder="Введите имя или ID узла для поиска" />

    <!-- Кнопка для выделения узла по имени или ID -->
    <button @click="highlightNode">Выделить узел</button>

    <!-- Область для отображения графа -->
    <div
      id="network"
      style="width: 100%; height: 400px; border: 1px solid #ccc"
    ></div>
  </div>
</template>

<style scoped>
#network {
  width: 100%;
  height: 400px;
  border: 1px solid #ccc;
}

.traceroute{
  width: 90%;
  margin: 0 auto 0 auto;
}
</style>
