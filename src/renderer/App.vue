<template>
  <div id="app">
    <!-- Bestehender Timer Content -->
    <transition name="slide">
      <div v-if="currentView === 'timer'" class="timer-view">
        <!-- HIER IST DER BESTEHENDE TIMER CODE -->
        <!-- Füge diesen Todo-Button irgendwo im Timer-Bereich hinzu: -->
        <button @click="showTodoList" class="todo-toggle-btn">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M9 11l3 3L22 4"></path>
            <path d="M21 12v7a2 2 0 01-2 2H5a2 2 0 01-2-2V5a2 2 0 012-2h11"></path>
          </svg>
        </button>
      </div>
    </transition>

    <!-- Neue Todo-Liste View -->
    <transition name="slide">
      <todo-list v-if="currentView === 'todo'" @close="showTimer" />
    </transition>
  </div>
</template>

<script>
// Importiere die neue TodoList Komponente
import TodoList from './components/TodoList.vue'

export default {
  name: 'App',
  components: {
    TodoList,
    // ... andere bestehende Komponenten
  },
  data() {
    return {
      currentView: 'timer', // Neue data property
      // ... andere bestehende data properties
    }
  },
  methods: {
    showTodoList() {
      this.currentView = 'todo'
    },
    showTimer() {
      this.currentView = 'timer'
    },
    // ... andere bestehende methods
  },
  mounted() {
    // Keyboard shortcuts
    window.addEventListener('keydown', (e) => {
      // Pfeil rechts zeigt Todo-Liste
      if (e.key === 'ArrowRight' && this.currentView === 'timer') {
        this.showTodoList()
      }
      // Pfeil links oder Escape geht zurück zum Timer
      else if ((e.key === 'ArrowLeft' || e.key === 'Escape') && this.currentView === 'todo') {
        this.showTimer()
      }
    })
  }
}
</script>

<style>
/* Füge diese Styles zu deinem bestehenden Style-Block hinzu */

.todo-toggle-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: var(--color-background-light, rgba(255, 255, 255, 0.1));
  border: 1px solid var(--color-border, rgba(255, 255, 255, 0.2));
  border-radius: 4px;
  padding: 0.5rem;
  color: var(--color-foreground, #ecf0f1);
  cursor: pointer;
  transition: all 0.2s;
  z-index: 10;
}

.todo-toggle-btn:hover {
  background: var(--color-background-lighter, rgba(255, 255, 255, 0.2));
  transform: scale(1.05);
}

/* Slide Transitions für View-Wechsel */
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
  position: absolute;
  width: 100%;
  height: 100%;
}

.slide-enter {
  transform: translateX(100%);
}

.slide-leave-to {
  transform: translateX(-100%);
}

.timer-view {
  position: relative;
  width: 100%;
  height: 100%;
}
</style>
