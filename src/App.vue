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
        <h2 class="task-heading">Aufgaben</h2>
        <div class="search-group">
          <input
            v-model="searchTerm"
            placeholder="Suchen"
            class="search-input"
          />
        </div>
      </div>
      <div class="task-list-content">
        <div class="input-group">
          <input
            v-model="newTask"
            @keyup.enter="addTask"
            placeholder="Neue Aufgabe"
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

<style scoped>
  .pomodoro-timer {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    max-height: 100vh;
    overflow-y: hidden;
  }

  h1 {
    font-size: 3rem;
    margin-bottom: 1rem;
    color: #000;
  }

  .break-text {
    color: #ff3b30;
    font-weight: bold;
  }

  .button-group {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    margin-bottom: 20px;
  }

  .btn {
    font-size: 1rem;
    padding: 0.8rem 1.5rem;
    margin: 5px;
    border: none;
    border-radius: 10px;
    background-color: #007aff;
    color: #fff;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .btn:hover {
    background-color: #0056b3;
  }

  .btn:disabled {
    background-color: #e5e5e5;
    color: #a6a6a6;
    cursor: not-allowed;
  }

  .btn-danger {
    background-color: #ff3b30;
  }

  .btn-danger:hover {
    background-color: #cc1f1a;
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
    border-bottom-color: #007aff;
    color: #000;
  }

  .task-list {
    background-color: #fff;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
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
    color: #000;
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
    padding: 0.8rem;
    font-size: 1rem;
    border: none;
    border-radius: 5px;
    background-color: #f2f2f2;
  }

  .btn-add {
    background-color: #34c759;
  }

  .btn-add:hover {
    background-color: #2b9e46;
  }

  .task-item {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: space-between;
    padding: 0.5rem 0;
    border-bottom: 1px solid #e5e5e5;
  }

  .task-item:last-child {
    border-bottom: none;
  }

  .task-text {
    color: #000;
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
    padding: 0.8rem;
    font-size: 1rem;
    border: none;
    border-radius: 5px;
    background-color: #f2f2f2;
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
