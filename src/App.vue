<template>
  <div class="pomodoro-timer">
    <div class="switch-group">
      <span :class="{ active: !isBreak }" @click="switchToWork">Arbeit</span>
      <span :class="{ active: isBreak }" @click="switchToBreak">Pause</span>
    </div>

    <h1>{{ minutes }}:{{ seconds < 10 ? "0" : "" }}{{ seconds }}</h1>
    <p v-if="isBreak" class="break-text">Pause!</p>
    <div class="button-group">
      <button @click="startTimer" :disabled="isRunning" class="btn">
        {{ isBreak ? "Pause fortsetzen" : "Start" }}
      </button>
      <button @click="stopTimer" :disabled="!isRunning" class="btn">
        Stop
      </button>
      <button @click="resetTimer" class="btn">Reset</button>
    </div>

    <div class="task-list">
      <div class="task-list-header">
        <h2 class="task-heading">Aufgaben:</h2>
        <div class="search-group">
          <input
            v-model="searchTerm"
            placeholder="Suchen..."
            class="search-input"
          />
        </div>
      </div>
      <div class="task-list-content">
        <div class="input-group">
          <input
            v-model="newTask"
            @keyup.enter="addTask"
            placeholder="Neue Aufgabe..."
            class="task-input"
          />
          <button @click="addTask" class="btn btn-add">Hinzufügen</button>
        </div>
        <ul>
          <li
            v-for="(task, index) in filteredTasks"
            :key="index"
            class="task-item"
          >
            <span
              :class="{ completed: task.completed }"
              @click="toggleTaskCompletion(index)"
            >
              <span
                :class="{
                  strikethrough: task.completed,
                  'task-text': !task.completed,
                }"
                >{{ task.text }}</span
              >
            </span>
            <button @click="removeTask(index)" class="btn btn-danger">
              Löschen
            </button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        workMinutes: 25,
        breakMinutes: 5,
        minutes: 25,
        seconds: 0,
        isRunning: false,
        isBreak: false,
        timer: null,
        tasks: [],
        newTask: "",
        searchTerm: "",
      };
    },
    computed: {
      filteredTasks() {
        if (!this.searchTerm) {
          return this.tasks;
        }
        const lowerCaseSearchTerm = this.searchTerm.toLowerCase();
        return this.tasks.filter((task) =>
          task.text.toLowerCase().includes(lowerCaseSearchTerm)
        );
      },
    },
    methods: {
      startTimer() {
        this.isRunning = true;
        this.timer = setInterval(() => {
          if (this.seconds === 0) {
            if (this.minutes === 0) {
              this.stopTimer();
              this.switchToWork();
              return;
            }
            this.minutes--;
            this.seconds = 59;
          } else {
            this.seconds--;
          }
        }, 1000);
      },
      stopTimer() {
        this.isRunning = false;
        clearInterval(this.timer);
      },
      resetTimer() {
        this.minutes = this.isBreak ? this.breakMinutes : this.workMinutes;
        this.seconds = 0;
        this.isRunning = false;
        clearInterval(this.timer);
      },
      switchToWork() {
        this.isBreak = false;
        this.minutes = this.workMinutes;
        this.seconds = 0;
        this.stopTimer();
      },
      switchToBreak() {
        this.isBreak = true;
        this.minutes = this.breakMinutes;
        this.seconds = 0;
        this.stopTimer();
      },
      loadTasks() {
        if (localStorage.getItem("tasks")) {
          this.tasks = JSON.parse(localStorage.getItem("tasks"));
        }
      },
      saveTasks() {
        localStorage.setItem("tasks", JSON.stringify(this.tasks));
      },
      addTask() {
        if (this.newTask.trim() !== "") {
          this.tasks.push({
            text: this.newTask,
            completed: false,
          });
          this.newTask = "";
          this.saveTasks();
        }
      },
      removeTask(index) {
        this.tasks.splice(index, 1);
        this.saveTasks();
      },
      toggleTaskCompletion(index) {
        this.tasks[index].completed = !this.tasks[index].completed;
        this.saveTasks();
      },
    },
    mounted() {
      this.loadTasks();
    },
  };
</script>

<style scoped>
  .pomodoro-timer {
    font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f4f4f4;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    max-height: 100vh;
    overflow-y: hidden;
  }

  h1 {
    font-size: 3rem;
    margin-bottom: 1rem;
    color: #333;
  }

  .break-text {
    color: #ff5722;
    font-weight: bold;
  }

  .button-group {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    margin-bottom: 20px;
  }

  .btn {
    font-size: 1.1rem;
    padding: 0.5rem 1rem;
    margin: 5px;
    border: none;
    border-radius: 3px;
    background-color: #2196f3;
    color: #fff;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .btn:hover {
    background-color: #1976d2;
  }

  .btn:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }

  .btn-danger {
    background-color: #f44336;
  }

  .btn-danger:hover {
    background-color: #d32f2f;
  }

  .switch-group {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
  }

  .switch-group span {
    padding: 10px 20px;
    cursor: pointer;
    transition: background-color 0.3s, color 0.3s;
    border-bottom: 2px solid transparent;
    color: #888;
  }

  .switch-group span.active {
    border-bottom-color: #4caf50;
    background-color: #e0e0e0;
    color: #333;
  }

  .task-list {
    background-color: #fff;
    padding: 20px;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
    height: 400px;
  }

  .task-list-header {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
  }

  .task-heading {
    color: #333;
    font-weight: bold;
    margin-bottom: 10px;
  }

  .task-list-content {
    overflow-y: auto;
  }

  .input-group {
    display: flex;
    flex-wrap: wrap;
    margin-bottom: 10px;
    gap: 10px;
  }

  .task-input {
    flex: 1;
    padding: 0.5rem;
    font-size: 1.1rem;
    border: 1px solid #ccc;
    border-radius: 3px;
  }

  .btn-add {
    background-color: #4caf50;
  }

  .btn-add:hover {
    background-color: #45a049;
  }

  .task-item {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: space-between;
    padding: 0.5rem 0;
    border-bottom: 1px solid #eee;
  }

  .task-item:last-child {
    border-bottom: none;
  }

  .task-text {
    color: #333;
    font-weight: bold;
  }

  .completed {
    color: #888;
  }

  .strikethrough {
    text-decoration: line-through;
  }

  .search-group {
    flex: 1;
    margin-left: 20px;
  }

  .search-input {
    width: 100%;
    padding: 0.5rem;
    font-size: 1.1rem;
    border: 1px solid #ccc;
    border-radius: 3px;
  }

  @media (max-width: 600px) {
    .pomodoro-timer {
      padding: 10px;
    }

    h1 {
      font-size: 2rem;
    }

    .task-list {
      height: 300px;
    }

    .search-group {
      margin-left: 0;
      margin-top: 10px;
    }
  }
</style>
