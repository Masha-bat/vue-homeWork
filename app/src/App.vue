<template>
  <div id="app">
    <h1 class="title">{{ title }}</h1>

    <div class="todo">
      <input v-model="todo" type="text" placeholder="Task name" />
      <input v-model="date" type="date" placeholder="Select date" />
      <p></p>

      <input v-model="category" type="text" placeholder="Category" />
      <button @click="addTodo">Add Task</button>
      <hr />

      <div class="filters">
        <input v-model="searchQuery" type="text" placeholder="Search by name or category" />
        <p></p>
        <input v-model="filterDate" type="date" placeholder="Filter by date" />
        <p></p>
        <select v-model="filterStatus">
          <option value="">All</option>
          <option value="completed">Completed</option>
          <option value="notCompleted">Not Completed</option>
        </select>
        <label>
          <input type="checkbox" v-model="filterWithoutDate" />
          Без даты
        </label>
      </div>

      <hr />
      <div v-for="(todo, i) in filteredTodos" :key="todo.id">
        <p>
          <span class="todo__id">{{ i + 1 }}. </span>
          <span
            class="todo__text"
            :class="{ todo__text__isShown: todo.isComplete }"
          >
            {{ todo.text }} - <em>{{ todo.category || 'No Category' }}</em>
          </span>
          <input v-model="todo.isComplete" class="todo__check" type="checkbox" @change="updateTodo(i)" />
          <button @click="removeToDo(i)">Удалить</button>
        </p>
        <p class="todo__date">{{ todo.date || 'No Date' }}</p>
      </div>
      <hr />
      <p class="todo__count">Tasks: {{ filteredTodos.length }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: "Todo app",
      todo: "",
      date: "",
      category: "",
      todos: [],
      id: 0,
      searchQuery: "",    // Для поиска по названию или категории
      filterDate: "",     // Для фильтрации по дате
      filterStatus: "",   // Для фильтрации по статусу
      filterWithoutDate: false, // Для фильтрации задач без даты
    };
  },
  computed: {
    filteredTodos() {
      return this.todos.filter((todo) => {
        // Фильтр по названию задачи или категории
        const matchesQuery =
          this.searchQuery === "" ||
          todo.text.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          (todo.category && todo.category.toLowerCase().includes(this.searchQuery.toLowerCase()));

        // Фильтр по дате (если указана)
        const matchesDate =
          !this.filterDate || todo.date === this.filterDate;

        // Фильтр по статусу выполнения задачи
        const matchesStatus =
          this.filterStatus === "" ||
          (this.filterStatus === "completed" && todo.isComplete) ||
          (this.filterStatus === "notCompleted" && !todo.isComplete);

        // Фильтр по отсутствию даты
        const matchesWithoutDate = !this.filterWithoutDate || !todo.date;

        return matchesQuery && matchesDate && matchesStatus && matchesWithoutDate;
      });
    },
  },
  async mounted() {
    const data = await localStorage.getItem("todos");
    if (data) {
      this.todos = JSON.parse(data);
      this.id = this.todos.length > 0 ? this.todos[this.todos.length - 1].id + 1 : 0;
    }
  },
  methods: {
    addTodo() {
      if (this.todo.trim() !== "") {
        this.todos.push({
          id: this.id++,
          text: this.todo,
          date: this.date || null,
          category: this.category.trim() || null,
          isComplete: false,
        });
        this.saveTodos(); // Сохраняем в локальное хранилище
        this.todo = "";
        this.date = "";
        this.category = "";
      }
    },
    updateTodo(index) {
      // Просто вызываем saveTodos для обновления локального хранилища
      this.saveTodos();
    },
    removeToDo(index) {
      this.todos.splice(index, 1);
      this.saveTodos();
    },
    saveTodos() {
      localStorage.setItem("todos", JSON.stringify(this.todos));
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.title {
  color: #a84849;
}

.todo {
  width: 300px;
  height: 100%;
  text-align: center;
  background: #a84848;
  padding: 30px;
  outline: 1px solid #2c3e50;
  margin: 0 auto;

  &__id {
    color: #ffffff;
    font-weight: bold;
  }

  &__text {
    font-size: 15px;
    color: #ffffff;
    text-transform: capitalize;

    &__isShown {
      color: #cccccc;
      text-decoration: line-through;
    }
  }

  &__date {
    display: block;
    font-size: 13px;
    color: #ddd;
  }

  &__check {
    display: inline-block;
    margin-left: 10px;
  }

  &__count {
    color: #ffffff;
    font-weight: bold;
  }
}
</style>
