<template>
  <div class="flex">
    <TodoFooter v-on:add-task="addTask()" v-bind:tasks="tasks" v-bind:totalTime="totalTime" />
    <Modal
      :active="modalActive"
      :pendingDelete="pendingDelete"
      v-on:close-modal="closeModal"
      v-on:confirm-delete="deleteTask"
    />

    <ul>

      <li v-for="(task, index) in pagedTasks" :data-index="index" v-bind:key="index">
        <input v-model="task.item" @keyup="saveTasks()" />
        <span
          class="timer"
          :class="{active: task.active}"
        >{{ (formattedElapsedTime(index)) ? formattedElapsedTime(index) : "00:00:00" }}</span>

        <button v-if="!task.active" @click="start(index)" :class="{ active: task.active }">
          <play-icon size="1.5x" class="custom-class active" :class="{ active: task.active }"></play-icon>
        </button>
        <button v-if="task.active" @click="stop(index)">
          <pause-icon size="1.5x" class="custom-class" :class="{ inactive: task.active }"></pause-icon>
        </button>

        <button
          @click="openModal(index)"
          style="margin-left:5px;margin-right:5px;"
          v-on:confirm-delete="deleteTask"
        >
          <trash-2-icon size="1.5x" class="custom-class inactive"></trash-2-icon>
        </button>
      </li>
            <div v-if="pages > 1" id="pages">
        <span v-for="i in pages" v-bind:key="i" :class="{ active : i == currentPage, 'pagination' : i }">
          <a @click="setCurrentPage(i)">{{i}}</a>
        </span>
      </div>
    </ul>
  </div>
</template>

<script>
import { PauseIcon, PlayIcon, Trash2Icon } from "vue-feather-icons";
import * as workerTimers from "worker-timers";
import TodoFooter from "./TodoFooter.vue";
import Modal from "./Modal.vue";

export default {
  name: "TodoList",
  data: function() {
    return {
      modalActive: false,
      pendingDelete: null,
      pageSize: 6,
      currentPage: 1
    };
  },
  components: {
    PlayIcon,
    Trash2Icon,
    PauseIcon,
    TodoFooter,
    Modal
  },
  props: ["tasks", "totalTime"],
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
      this.modalActive = true;
      this.pendingDelete = index;
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
.flex {
  flex-direction: column;
  background: white;
  box-shadow: 2px 2px 5px 1px gainsboro;
  border-radius: 5px;
  padding-right: 20px;
  padding-top: 20px;
  padding-bottom: 20px;
  position: relative;
}
li {
  position: relative;
}
.timer {
  margin-left: 10px;
  margin-right: 10px;
  background: #efefef;
  border: 1px solid white;
  border-left-color: white;
  border-left-style: solid;
  border-left-width: 1px;
  padding: 8.5px 10px;
  border-left: 1px solid gainsboro;
  position: absolute;
  right: 114px;
  top: 0px;
  border-top-right-radius: 6px;
  border-bottom-right-radius: 6px;
}
.timer.active {
  background: #aacc00;
  color: white;
  border: 1px solid #aacc00;
}
.pagination {
  border: 1px solid #aacc00;
  color:white;
  background: #aacc00;
  margin-right: 5px;
  cursor: pointer;
}
.pagination a {
    padding: 20px 10px;

}
.pagination.active {
  background: #e63946;
  border: 1px solid #e63946;
}
#pages {
  position: absolute;
  bottom: 10px;
  left: 90px;
}
</style>