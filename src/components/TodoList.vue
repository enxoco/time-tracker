<template>
  <div>
    <!-- <TodoFooter v-on:add-task="addTask()" v-bind:tasks="tasks" v-bind:totalTime="totalTime" /> -->


    <ul class="task-list">
    <Modal
      :active="modalActive"
      :pendingDelete="pendingDelete"
      v-on:close-modal="closeModal"
      v-on:confirm-delete="deleteTask"
      v-on:toggle-modal="toggleModalSetting"
    />
      <li class="task" v-for="(task, index) in pagedTasks" :data-index="index" v-bind:key="index">
        <input v-model="task.item" @keyup="saveTasks()" />
        <span
          class="input-timer"
          :class="{active: task.active}"
        >{{ (formattedElapsedTime(index)) ? formattedElapsedTime(index) : "00:00:00" }}</span>

          <a v-if="!task.active" @click="start(index)" class="start-button" :class="{ active: task.active }"></a>
          <a v-if="task.active" @click="stop(index)" class="stop-button" :class="{ active: task.active }"></a>

          <a @click="openModal(index)" v-on:confirm-delete="deleteTask" class="delete-button"></a>

      </li>
            <div v-if="pages > 1" id="pages">
        <span v-for="i in pages" v-bind:key="i" :class="{ active : i == currentPage, 'pagination' : i }" @click="setCurrentPage(i)">
          <a >{{i}}</a>
        </span>
      </div>
    </ul>
  </div>
</template>

<script>
// import { Trash2Icon } from "vue-feather-icons";
import * as workerTimers from "worker-timers";
// import TodoFooter from "./TodoFooter.vue";
import Modal from "./Modal.vue";

export default {
  name: "TodoList",
  data: function() {
    return {
      modalActive: false,
      pendingDelete: null,
      pageSize: 6,
      currentPage: 1,
      dismissModal: false
    };
  },
  components: {
    // Trash2Icon,
    // PauseIcon,
    // TodoFooter,
    Modal
  },
  props: ["tasks"],
  methods: {
    addTask() {
      this.tasks.push({ item: null, timer: 0, elapsedTime: 0, active: false });
      this.saveTasks();
    },
    saveTasks() {
      const parsed = JSON.stringify(this.tasks);
      localStorage.setItem("tasks", parsed);
      this.pendingDelete = null;
      this.modalActive = false;
    },


    formattedElapsedTime(index) {
      if (this.tasks[index].elapsedTime) {
        const date = new Date(null);
        date.setSeconds(this.tasks[index].elapsedTime / 1000);
        const utc = date.toUTCString();
        return utc.substr(utc.indexOf(":") - 2, 8);
      }
    },
    start(index) {
      if (this.tasks[index].active) {
        return;
      }
      this.tasks[index].timer = workerTimers.setInterval(() => {
        this.tasks[index].elapsedTime += 1000;
        this.saveTasks();
        this.getTotalTime();
      }, 1000);
      this.tasks[index].active = true;
    },
    stop(index) {
      workerTimers.clearInterval(this.tasks[index].timer);
      this.tasks[index].active = false;
      this.saveTasks();
    },
    reset(index) {
      this.tasks[index].elapsedTime = 0;
      this.tasks[index].active = false;
      this.saveTasks();
    },
    openModal(index) {
              this.pendingDelete = index;

      if (this.dismissModal === false) {
        this.modalActive = true;
      } else {
        this.deleteTask()
      }

    },
    deleteTask() {
      this.modalActive = !this.modalActive;
      this.tasks.splice(this.pendingDelete, 1);
      this.saveTasks();
    },
    getTotalTime() {
      this.totalTime = 0;
      this.tasks.forEach(task => {
        this.totalTime += task.elapsedTime;
      });
      const date = new Date(null);
      date.setSeconds(this.totalTime / 1000);
      const utc = date.toUTCString();
      this.totalTime = utc.substr(utc.indexOf(":") - 2, 8);
    },
    closeModal() {
      console.log("not deleting");
      this.modalActive = !this.modalActive;
    },
    setCurrentPage(page) {
      this.currentPage = page;
    },
            toggleModalSetting() {
          this.dismissModal = !this.dismissModal
        }
  },
  computed: {
    pagedTasks: function() {
      if (this.pages > 1) {
      var begin = (this.currentPage - 1) * this.pageSize;
      var end = begin + this.pageSize;

      return this.tasks.slice(begin, end);
      } else {
        return this.tasks.slice(0, this.pageSize)
      }

    },
    pages: function() {
      return Math.ceil(this.tasks.length / this.pageSize);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.start-button {
    background-image: url('/time-tracker/dist/img/startIcon.png');
    height: 70px;
    width: 70px;
    display: inline-block;
    background-repeat: no-repeat;
    border: 5px solid #222831;
    border-radius: 50%;
    background-position: center center;
    background-color: #393E46;
    cursor: pointer;
    margin-left: 30px;
}
.start-button:hover {
    background-image: url('/time-tracker/dist/img/startIcon.png');
    height: 70px;
    width: 70px;
    display: inline-block;
    background-repeat: no-repeat;
    border: 5px solid #222831;
    border-radius: 50%;
    background-position: center center;
    background-color: white;
    cursor: pointer;
}
.stop-button {
    background-image: url('/time-tracker/dist/img/stopIcon.png');
    height: 70px;
    width: 70px;
    display: inline-block;
    background-repeat: no-repeat;
    border: 5px solid #222831;
    border-radius: 50%;
    background-position: center center;
    background-color: #393E46;
    cursor: pointer;
    margin-left: 30px;
}
.delete-button {
    background-image: url('/time-tracker/dist/img/deleteIcon.png');
    height: 70px;
    width: 70px;
    display: inline-block;
    background-repeat: no-repeat;
    border: 5px solid #222831;
    border-radius: 50%;
    background-position: center center;
    background-color: #393E46;
    cursor: pointer;
}
.pagination {
    font-size: 32px;
    color: #32e0c4;
    background: white;
    width: 40px;
    height: 40px;
    display: inline-block;
    padding: 10px;
    margin-right: 10px;
    text-align: center;
    cursor: pointer;
    border-radius: 5px;
}
.pagination.active {
      background: #32e0c4;
    color: white;
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
}
@media (max-width: 425px) {
  .wrapper {
    padding: 20px;
  }
  .add-task-button {
    position: absolute;
    top: 20px;
    right: 20px;
  }
  .task input {
    padding-left: 20px;
    font-size: 16px;
  }
  .start-button, .stop-button, .delete-button {
    height: 30px;
    width: 30px;
    background-size: 75%;
    border:none;
  }
  .start-button {
    margin-left: 5px;
    margin-right: 5px;
  }
  .start-button:hover, .stop-button:hover, .delete-button:hover {
    height: 30px;
    width: 30px;
    background-size: 75%;
    border:none;
  }
  }

</style>