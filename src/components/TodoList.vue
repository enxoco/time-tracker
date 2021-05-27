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
      <li class="task" v-for="(task, index) in pagedTasks" :data-index="index" v-bind:key="index" :class="{visible: task.visible}">
        <!-- <span class="button suggestion-button">Recent<br>Tasks</span> -->
        <input class="client" v-model="task.item" @change="saveTasks()" placeholder="client / project" @keyup="saveTasks()"/>
        <input class="input-code" type="text" v-model="task.code" placeholder="project code" @keyup="saveTasks()"/>

        <input type="text" class="input-timer" :class="{active: task.active}" :data-index="index" :data-elapsed="task.elapsedTime" :value="formattedElapsedTime(index)" @change="convertHourToSeconds($event, index)"/>
        <div class="controls">
        <a
          v-if="!task.active"
          @click="start(index)"
          class="start-button"
          :class="{ active: task.active }"
        ></a>
        <a
          v-if="task.active"
          @click="stop(index)"
          class="stop-button"
          :class="{ active: task.active }"
        ></a>

        <a @click="openModal(index)" v-on:confirm-delete="deleteTask" class="delete-button"></a>
        </div>
      </li>

      <div v-if="pages > 1" id="pages">
        <span
          v-for="i in pages"
          v-bind:key="i"
          :class="{ active : i == currentPage, 'pagination' : i }"
          @click="setCurrentPage(i)"
        >
          <a>{{i}}</a>
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
// import { VueAutosuggest } from 'vue-autosuggest';

export default {
  name: "TodoList",
  data: function() {
    return {
      modalActive: false,
      pendingDelete: null,
      pageSize: 99,
      currentPage: 1,
      indexOffset: 0,
      dismissModal: false,
      selectedTaskId: 0,
      clients: [
        '1st Franklin',
        'Cadence',
        'ES Kluft',
        'Hooters',
        'Hoots',
        'ICOM',
        'Morrison',
        'MYCELX',
        'Palmetto Dunes',
        'Raceway',
        'Tosca',
        'Triad'
      ]
    };
  },
  components: {
    // VueAutosuggest,
    Modal
  },
  props: ["tasks", "selectedDay", "taskHeadings", "weekOffset"],

  methods: {
    saveTasks() {
      const parsed = JSON.stringify(this.tasks);
      localStorage.setItem("tasks", parsed);
      localStorage.setItem("dismissModal", this.dismissModal);
      this.pendingDelete = null;
      this.modalActive = false;
    },
    convertHourToSeconds(event, index) {
    let [h, m, s] = event.target.value.split(':') // split it at the colons

    var totalSeconds = (h) * 60 * 60 + (+m) * 60 + (+s);
    this.tasks[index].elapsedTime = +totalSeconds
    this.saveTasks()
    },
    formattedElapsedTime(index) {
      const date = new Date(null);
      date.setSeconds(this.tasks[index].elapsedTime);
      const utc = date.toISOString()
      return utc.substr(11,8)
      // return date.setSeconds(this.tasks[index].elapsedTime).toISOString()
      // return date.toISOString().substr(11,8)
    },
    start(index) {
      if (this.currentPage != 1) {
        index = index + this.pageSize * (this.currentPage - 1);
      }
      if (this.tasks[index].active) {
        return;
      }
      this.tasks[index].timer = workerTimers.setInterval(() => {
        this.tasks[index].elapsedTime += 1;
        this.saveTasks();
        this.getTotalTime();
      }, 1000);
      this.tasks[index].active = true;
    },
    stop(index) {
      if (this.currentPage != 1) {
        index = index + this.pageSize * (this.currentPage - 1);
      }
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
      if (this.currentPage != 1) {
        // 1 + 5 + 2 -1
        index = index + this.pageSize * (this.currentPage - 1);
      }
      this.pendingDelete = index;

      if (this.dismissModal === false) {
        this.modalActive = true;
      } else {
        this.deleteTask();
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
      this.dismissModal = !this.dismissModal;
    },
    getTodaysDate(){
      console.log('day', new Date())
      return new Date()
    },
    autoSuggestTask(event){
      if (event.target.value == ""){
        event.target.value = JSON.stringify(this.taskHeadings)
      }
      
    }

  },
  mounted() {
    localStorage.getItem("dismissModal")
      ? (this.dismissModal = true)
      : (this.dismissModal = false);

    console.log("dismiss ", this.dismissModal);
  },
  computed: {
    pagedTasks: function() {
      let index = this.selectedDay
      var curr = new Date; // get current date
      var first = (curr.getDate() - this.weekOffset) - curr.getDay() + +index; // First day is the day of the month - the day of the week
      let [month, day] = new Date(curr.setDate(first)).toLocaleDateString("en-US").split("/");
      this.tasks.map(task => {
        if (task.date == `${month}/${day}`){
          task.visible = true
        } else {
          task.visible = false
        }
      })
      return this.tasks
     
    },

    pages: function() {
      return Math.ceil(this.tasks.length / this.pageSize);
    },
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.start-button {
  background-image: url("/time-tracker/dist/img/fi-rr-play.svg");
  background-size: 35px 35px;
  height: 50px;
  width: 50px;
  background-repeat: no-repeat;
  background-position: center;
  border: 2px solid #eeeeee;
  border-radius: 50%;
  background-color: lightgreen;
  cursor: pointer;
  margin-left: 30px;
    background-size: 35px;
  background-position: center;
}
.start-button:hover {
  background-image: url("/time-tracker/dist/img/fi-rr-play.svg");
  height: 50px;
  width: 50px;
  display: inline-block;
  background-repeat: no-repeat;
  border: 2px solid #eeeeee;
  border-radius: 50%;
  background-position: center center;
  background-color: darkseagreen;
  cursor: pointer;
}
.stop-button {
  background-image: url("/time-tracker/dist/img/fi-rr-stop.svg");
  height: 50px;
  width: 50px;
  display: inline-block;
  background-repeat: no-repeat;
  background-size: 25px;
  background-position: center;
  border: 2px solid #eeeeee;
  border-radius: 50%;
  background-color: lightsalmon;
  cursor: pointer;
  margin-left: 30px;
}
.delete-button {
  background-image: url("/time-tracker/dist/img/fi-rr-cross-small.svg");
  height: 50px;
  width: 50px;
  display: inline-block;
  background-repeat: no-repeat;
  border: 2px solid #eeeeee;
  border-radius: 50%;
  background-position: center center;
  background-color: lightsalmon;
  cursor: pointer;
}
.delete-button:hover {
  background-color: darksalmon;
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
.task {
  display: none;
}
.task.visible {
    list-style: none;
    position: relative;
    display: grid;
    grid-template-columns: 60% 11% 18% 14%;
    margin-bottom: 10px;
    box-shadow: 1px 6px 8px rgb(57 62 70 / 40%);
    border-radius: 10px;
    background: white;
    border-top-right-radius: 50px;
    border-bottom-right-radius: 50px;
    padding: 10px;
}
.task.visible input {
  outline: none;
  width: 100%;
  border: 0px;
  border-radius: 10px;
  font-size: 18px;
  font-weight: 400;
  padding: 0px;
  margin-right: 5px;
}
.input-date {
  pointer-events: none;
  background: gainsboro;
  text-align: center;
}
.input-client {
  padding-left: 5px;
}
/* span.client-suggestions {
  background: gainsboro;
  color: black;
  list-style: none;
} */
div#autosuggest-autosuggest__results {
    background: #eeeeee;
    position: absolute;
    z-index: 9;
    padding: 10px 20px 20px 0;
    padding-top: 10px;
    padding-right: 20px;
    padding-bottom: 20px;
    padding-left: 0px;
}
/* .task.visible span {
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px;
  background: #32e0c4;
  color: white;
  padding: 30px 20px;
} */

.autosuggest {
  position: absolute;
  top: 50px;
  background: lightgreen;
  padding: 50px;
  z-index: 9;
  color: white;
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
  .start-button,
  .stop-button,
  .delete-button {
    height: 30px;
    width: 30px;
    background-size: 75%;
    border: none;
  }
  .start-button {
    margin-left: 5px;
    margin-right: 5px;
  }
  .start-button:hover,
  .stop-button:hover,
  .delete-button:hover {
    height: 30px;
    width: 30px;
    background-size: 75%;
    border: none;
  }
}
.controls {
  display: flex;
  right: 0;
  position: relative;
}
.button {
  background: lightsteelblue;
  border-radius: 10px;
  font-size: 50%;
  height: 20px;
  color: white;
  text-align: center;
  padding: 10px;
}
</style>