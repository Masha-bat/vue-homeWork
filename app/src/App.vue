<template>
  <div id="app" class="container">
    <h1 class="title">{{ title }}</h1>

    <div class="box is-centered has-background-pink custom-box">
      <div class="field is-grouped is-grouped-centered">
        <div class="control">
          <input
            v-model="todo"
            type="text"
            placeholder="Название задачи"
            class="input is-small"
            style="max-width: 300px;"
          />
        </div>
        <div class="control">
          <input
            v-model="category"
            type="text"
            placeholder="Категория"
            class="input is-small"
            style="max-width: 150px;"
          />
        </div>
        <div class="control">
          <input
            v-model="date"
            type="date"
            placeholder="Дата задачи"
            class="input is-small"
            style="max-width: 150px;"
          />
        </div>
        <div class="control">
          <button @click="addTodo" class="button is-light is-small">Добавить</button>
        </div>
      </div>

      <hr />

      <div class="filters">
        <div class="field is-grouped is-grouped-centered">
          <div class="control">
            <input
              v-model="searchQuery"
              type="text"
              placeholder="Поиск по названию или категории"
              class="input is-small"
              style="max-width: 200px;"
            />
          </div>
          <div class="control">
            <div class="select is-small">
              <select v-model="filterCategory">
                <option value="">Все категории</option>
                <option v-for="cat in uniqueCategories" :key="cat" :value="cat">{{ cat }}</option>
              </select>
            </div>
          </div>
          <div class="control">
            <div class="select is-small">
              <select v-model="filterStatus">
                <option value="">Все</option>
                <option value="completed">Выполненные</option>
                <option value="notCompleted">Невыполненные</option>
              </select>
            </div>
          </div>
          <div class="control">
            <input
              v-model="filterDate"
              type="date"
              placeholder="Фильтр по дате"
              class="input is-small"
              style="max-width: 150px;"
            />
          </div>
        </div>
      </div>

      <hr />

      <div v-for="(todo, i) in filteredTodos" :key="todo.id" class="box">
        <div class="columns is-vcentered">
          <div class="column is-narrow">
            <span>{{ i + 1 }}.</span>
          </div>

          <div class="column">
            <span v-if="!todo.isEditing" class="todo__text" 
              :class="{ 'has-text-grey-light': todo.isComplete }">
                {{ todo.text }}
            </span>
            <div v-else>
              <input v-model="todo.text" class="input is-small" />
              <input v-model="todo.category" class="input is-small" />
              <input v-model="todo.date" type="date" class="input is-small" />
            </div>
          </div>

          <div class="column is-narrow">
            <input
              v-model="todo.isComplete"
              type="checkbox"
              class="checkbox"
              @change="updateTodo"
            />
          </div>

          <div class="column is-narrow">
            <button @click="editTodo(todo)" class="button is-small is-light">{{ todo.isEditing ? 
            'Сохранить' : 'Редактировать' }}</button>
            <button @click="removeToDo(i)" class="button is-small is-danger">Удалить</button>
          </div>
        </div>

        <div class="todo__info">
          <span class="category">{{ todo.category }}</span> <span class="date">{{ todo.date }}</span>
        </div>
      </div>

      <p class="has-text-white">Задач: {{ filteredTodos.length }}</p>
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
      searchQuery: "",
      filterDate: "",
      filterStatus: "",
      filterCategory: "", 
    };
  },

  //Вычисляемые свойства
  computed: {
    uniqueCategories() {
      const categories = this.todos.map(todo => todo.category).filter(Boolean);
      return [...new Set(categories)];
    },
    
    filteredTodos() {
      return this.todos.filter((todo) => {
        const matchesQuery =
          this.searchQuery === "" ||
          todo.text.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          (todo.category && todo.category.toLowerCase().includes(this.searchQuery.toLowerCase()));

        const matchesCategory =
          !this.filterCategory || todo.category === this.filterCategory;

        const matchesDate =
          !this.filterDate || todo.date === this.filterDate || (this.filterDate === "" && !todo.date);

        const matchesStatus =
          this.filterStatus === "" ||
          (this.filterStatus === "completed" && todo.isComplete) ||
          (this.filterStatus === "notCompleted" && !todo.isComplete);

        return matchesQuery && matchesCategory && matchesDate && matchesStatus;
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
          isEditing: false, // Флаг редактирования
        });
        localStorage.setItem("todos", JSON.stringify(this.todos));
        this.todo = "";
        this.date = "";
        this.category = "";
      }
    },

    editTodo(todo) {
      if (todo.isEditing) {
        // Сохранить изменения
        localStorage.setItem("todos", JSON.stringify(this.todos));
      }
      todo.isEditing = !todo.isEditing; // Переключить режим редактирования
    },

    updateTodo() {
      localStorage.setItem("todos", JSON.stringify(this.todos));
    },

    removeToDo(index) {
      this.todos.splice(index, 1);
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
  margin-top: 20px;
  margin-bottom: 20px;
}

.title {
  color: #f14668; 
}

.box.has-background-pink {
  background-color: #ff5174; 
}

.filters {
  margin-bottom: 1.5rem;
}

.todo__date {
  font-size: 13px;
  color: #7a7a7a;
}

.todo__info {
  font-size: 0.9rem;
  color: #7a7a7a;
  text-transform: capitalize; 
}

.custom-box {
  width: 600px; /* Установите желаемую ширину */
  margin: 0 auto; /* Центрирование box */
}

.category,
.date {
  display: inline; /* Категория и дата в одной строке */
}
</style>
