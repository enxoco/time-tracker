<template>
  <div class="container">
    <div class="c-task-pane">
        <select @change="changeDate($event)">
            <option v-for="(date, index) in dates" :key="index" :value="index">{{ date }}</option>
        </select>
      <br />
      <select class="c-task-pane--select-box" @change="trackTask($event)">
        <option
          v-for="(task, index) in taskList"
          :key="index"
          :value="task.name"
        >
          {{ task.name }}
        </option>
        <option>Add New Project</option>
      </select>
      <div class="controls" v-if="selectedTask">
        <a
          v-if="!selectedTask.active"
          @click="start(index)"
          class="start-button"
          :class="{ active: selectedTask.active }"
        ></a>
        <a
          v-if="selectedTask.active"
          @click="stop(index)"
          class="stop-button"
          :class="{ active: selectedTask.active }"
        ></a>
        <span v-if="selectedTask[0]">{{ formattedElapsedTime(selectedTask[0]) }}</span>
        <!-- <a @click="openModal(index)" v-on:confirm-delete="deleteTask" class="delete-button">Delete</a> -->
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: ["tasks", "dates"],
  data() {
    return {
      selectedTask: null,
    };
  },
  methods: {
    changeDate(event){
        this.$emit('updateDay', event.target.value)
    },
    addTask() {
      console.log("tasky");
    },
    getTodaysDate() {
      let [month, day] = new Date().toLocaleDateString("en-US").split("/");

      return `${month}/${day}`;
    },
    trackTask(task) {
      console.log("tracky tasky", task.target.value);
      let selected = this.tasks.filter(
        (t) => t.item === task.target.value && t.date == this.getTodaysDate()
      );
      if (!selected)
      this.$emit('addQuickTask', task.target.value, )
      this.selectedTask = selected
        ? selected
        : {
            'active': false,
            'date': this.getTodaysDate(),
            'elapsedTime': 0,
            'item': task.target.value,
          };
    },
    getUniqueArray(arr = [], compareProps = []) {
      //https://medium.com/@jithinsebastian2/remove-duplicate-objects-from-an-array-of-objects-javascript-es6-9d36f705d376
      let modifiedArray = [];
      if (compareProps.length === 0 && arr.length > 0)
        compareProps.push(...Object.keys(arr[0]));
      arr.map((item) => {
        if (modifiedArray.length === 0) {
          modifiedArray.push(item);
        } else {
          if (
            !modifiedArray.some((item2) =>
              compareProps.every(
                (eachProps) => item2[eachProps] === item[eachProps]
              )
            )
          ) {
            modifiedArray.push(item);
          }
        }
      });
      return modifiedArray;
    },
    formattedElapsedTime(task) {
      const date = new Date(null);
      date.setSeconds(task.elapsedTime);
      const utc = date.toISOString();
      return utc.substr(11, 8);
      // return date.setSeconds(this.tasks[index].elapsedTime).toISOString()
      // return date.toISOString().substr(11,8)
    },
  },
  computed: {
    taskList: function () {
      return this.getUniqueArray(
        this.tasks.map((task) => {
          return { name: task.item, code: task.code, date: task.date };
        }),
        ["name"]
      ).filter(task => task.date === this.getTodaysDate())
    },
  },
};
</script>
<style scoped>
.container {
  max-width: 800px;
  background: #eee;
  display: flex;
  margin: 0 auto;
}
.c-task-pane {
  background: white;
  padding: 50px 20px;
  display: flex;
  flex-direction: column;
  text-align: center;
}

select.c-task-pane--select-box {
  background: #eee;
  border: none;
  padding: 5px 10px;
}
</style>