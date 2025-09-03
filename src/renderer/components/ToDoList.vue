<template>
  <div class="todo-container">
    <div class="todo-header">
      <h2>Todo List</h2>
      <button @click="$emit('close')" class="close-btn">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <line x1="18" y1="6" x2="6" y2="18"></line>
          <line x1="6" y1="6" x2="18" y2="18"></line>
        </svg>
      </button>
    </div>

    <div class="todo-input-wrapper">
      <input
        v-model="newTodo"
        @keyup.enter="addTodo"
        class="todo-input"
        placeholder="Add a new task..."
        ref="todoInput"
      />
      <button @click="addTodo" class="add-btn" :disabled="!newTodo.trim()">
        Add
      </button>
    </div>

    <div class="todo-list">
      <transition-group name="todo-item" tag="div">
        <div
          v-for="todo in todos"
          :key="todo.id"
          class="todo-item"
          :class="{ completed: todo.completed }"
        >
          <input
            type="checkbox"
            :id="`todo-${todo.id}`"
            v-model="todo.completed"
            @change="updateTodos"
            class="todo-checkbox"
          />
          <label :for="`todo-${todo.id}`" class="todo-label">
            {{ todo.text }}
          </label>
          <button @click="removeTodo(todo.id)" class="delete-btn">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="3 6 5 6 21 6"></polyline>
              <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
            </svg>
          </button>
        </div>
      </transition-group>

      <div v-if="todos.length === 0" class="empty-state">
        No tasks yet. Add one above!
      </div>
    </div>

    <div class="todo-footer" v-if="todos.length > 0">
      <span class="todo-count">
        {{ incompleteTodos }} of {{ todos.length }} remaining
      </span>
      <button
        v-if="completedTodos > 0"
        @click="clearCompleted"
        class="clear-completed"
      >
        Clear completed
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TodoList',
  data() {
    return {
      todos: [],
      newTodo: ''
    }
  },
  computed: {
    incompleteTodos() {
      return this.todos.filter(todo => !todo.completed).length
    },
    completedTodos() {
      return this.todos.filter(todo => todo.completed).length
    }
  },
  mounted() {
    this.loadTodos()
    this.$refs.todoInput.focus()
  },
  methods: {
    loadTodos() {
      // Load from electron store or localStorage
      const stored = localStorage.getItem('pomotroid-todos')
      if (stored) {
        try {
          this.todos = JSON.parse(stored)
        } catch (e) {
          console.error('Failed to load todos:', e)
          this.todos = []
        }
      }
    },
    saveTodos() {
      // Save to electron store or localStorage
      localStorage.setItem('pomotroid-todos', JSON.stringify(this.todos))
    },
    addTodo() {
      if (!this.newTodo.trim()) return
      
      this.todos.push({
        id: Date.now(),
        text: this.newTodo.trim(),
        completed: false,
        createdAt: new Date().toISOString()
      })
      
      this.newTodo = ''
      this.saveTodos()
    },
    removeTodo(id) {
      this.todos = this.todos.filter(todo => todo.id !== id)
      this.saveTodos()
    },
    updateTodos() {
      this.saveTodos()
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed)
      this.saveTodos()
    }
  }
}
</script>

<style scoped>
.todo-container {
  width: 100%;
  height: 100vh;
  padding: 2rem;
  background: var(--color-background, #2c3e50);
  color: var(--color-foreground, #ecf0f1);
  display: flex;
  flex-direction: column;
}

.todo-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.todo-header h2 {
  font-size: 2rem;
  font-weight: 300;
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  color: var(--color-foreground-secondary, #95a5a6);
  cursor: pointer;
  padding: 0.5rem;
  transition: color 0.2s;
}

.close-btn:hover {
  color: var(--color-foreground, #ecf0f1);
}

.todo-input-wrapper {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.todo-input {
  flex: 1;
  padding: 0.75rem;
  background: var(--color-background-light, #34495e);
  border: 1px solid var(--color-border, #4a5f7a);
  border-radius: 4px;
  color: var(--color-foreground, #ecf0f1);
  font-size: 1rem;
  transition: border-color 0.2s;
}

.todo-input:focus {
  outline: none;
  border-color: var(--color-accent, #3498db);
}

.todo-input::placeholder {
  color: var(--color-foreground-secondary, #7f8c8d);
}

.add-btn {
  padding: 0.75rem 1.5rem;
  background: var(--color-accent, #3498db);
  border: none;
  border-radius: 4px;
  color: white;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.2s;
}

.add-btn:hover:not(:disabled) {
  background: var(--color-accent-hover, #2980b9);
}

.add-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.todo-list {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 1rem;
}

.todo-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: var(--color-background-light, #34495e);
  border-radius: 4px;
  margin-bottom: 0.5rem;
  transition: all 0.3s;
}

.todo-item:hover {
  background: var(--color-background-lighter, #3d566e);
}

.todo-item.completed {
  opacity: 0.6;
}

.todo-item.completed .todo-label {
  text-decoration: line-through;
  color: var(--color-foreground-secondary, #7f8c8d);
}

.todo-checkbox {
  width: 20px;
  height: 20px;
  cursor: pointer;
}

.todo-label {
  flex: 1;
  cursor: pointer;
  user-select: none;
  word-break: break-word;
}

.delete-btn {
  background: none;
  border: none;
  color: var(--color-danger, #e74c3c);
  cursor: pointer;
  padding: 0.25rem;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.delete-btn:hover {
  opacity: 1;
}

.empty-state {
  text-align: center;
  padding: 3rem;
  color: var(--color-foreground-secondary, #7f8c8d);
  font-style: italic;
}

.todo-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1rem;
  border-top: 1px solid var(--color-border, #4a5f7a);
}

.todo-count {
  color: var(--color-foreground-secondary, #7f8c8d);
  font-size: 0.9rem;
}

.clear-completed {
  background: none;
  border: none;
  color: var(--color-danger, #e74c3c);
  cursor: pointer;
  font-size: 0.9rem;
  padding: 0.25rem 0.5rem;
  transition: opacity 0.2s;
}

.clear-completed:hover {
  opacity: 0.8;
}

/* Transitions */
.todo-item-enter-active,
.todo-item-leave-active {
  transition: all 0.3s;
}

.todo-item-enter {
  opacity: 0;
  transform: translateX(-30px);
}

.todo-item-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* Scrollbar styling */
.todo-list::-webkit-scrollbar {
  width: 8px;
}

.todo-list::-webkit-scrollbar-track {
  background: var(--color-background, #2c3e50);
}

.todo-list::-webkit-scrollbar-thumb {
  background: var(--color-foreground-secondary, #7f8c8d);
  border-radius: 4px;
}

.todo-list::-webkit-scrollbar-thumb:hover {
  background: var(--color-foreground, #95a5a6);
}
</style>
