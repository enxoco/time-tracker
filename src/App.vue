<template>
 <div class="wrapper" id="app">
   <div class="header">
     <span>
       <h1>Work Tracker</h1>
       <p>Your To-Do list and time tracking simplified</p>
     </span>
     <span class="c-button__control add-tasks" v-on:click="addTask">&#43;</span>
     <span class="c-button__control export-tasks" v-on:click="exportTasks" v-if="view === 'tasks'"></span>
     <span class="c-button__control view-timesheet" v-on:click="view = 'tasks'" v-if="view === 'timesheet'">&larr;</span>

   </div>
   <ul class="day-tabs" v-if="view === 'tasks'">
   <li class="day-tab" v-for="(day, index) in days" :class="{selected: selectedDay == (index +1)}" :key="index" @click="selectedDay = (index + 1)">
     {{ day }}
     <span class="totalTime">{{getDailyTime(index + 1)}}</span>
   </li>

   <li class="day-tab"><span>Total Time: {{getTotalTime()}}</span></li>
   </ul>
      <TodoList :tasks="tasks" :selectedDay="selectedDay" :taskHeadings="taskHeadings" v-if="view === 'tasks'"/>


      <Timesheet :tasks="tasks" :dateCodes="dateCodes" :timesheet="timesheet" v-if="view === 'timesheet'" />
            <span class="copyright">Copyright 2020 <a href="https://iliveinaterminal.com">Mike Conrad</a>.  Part of <a href="https://theopensuite.com">The Open Suite</a><a href="https://github.com/enxoco/time-tracker" style="float:right;">Get the code</a></span>

 </div>
</template>

<script>

/*
* TODOS
* 1 - Add an autosuggest feature to the client/project field.  Map through existing tasks for the week and use them
*     to populate the autosuggest.  This way you be sure to always use the same name for a project, you can save some
*     typing and it could autofill the project code.  Basically have the groundwork for this already in the taskHeadings
*     variable in the export function.
*
*/

import TodoList from "./components/TodoList.vue";
import Timesheet from "./components/TimeSheet.vue"
export default {
  name: "App",
  components: {
    TodoList,
    Timesheet
  },
  data: function() {
    return {
      tasks: [],
      totalTime: 0,
      days: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'],
      selectedDay: 2,
      taskIndex: 0,
      timesheet: [],
      dateCodes: [],
      taskHeadings: [],
      view: 'tasks'

    };
  },
  methods: {
    addTask() {
      this.tasks.push({
        item: null,
        timer: 0,
        elapsedTime: 0,
        active: false,
        id: this.taskIndex++,
        date: this.getTimeStamp(),
        visible: false,
        code: null,
      });
      this.saveTasks();
    },
    exportTasks(){
      

      //First we need to get a list of all unique tasks
      let days = []
      this.tasks.map(task => {
        days.push(task.date)
      })
      days = [...new Set(days)]
      //Second we need to split the tasks up by days.
      
      //Create an object to hold our timesheet data.
      let timesheet = {}
      //Map over our task names and make each one an object key
      this.taskHeadings.map(name => {
        timesheet[name] = []
      })
      //map over our tasks and taskheading assigning each task to its associated taskheading object
      this.tasks.map(task => {
        this.taskHeadings.map((name) => {
          if (task.item === name){
            timesheet[name] = {name, code: task.code, dates: [], tasks: []}
          }
        }
        )
      })

      //Get days/times associated with a task
      this.taskHeadings.map(name => {
        timesheet[name].tasks = this.tasks.filter(item => item.item === name)
      })
      this.timesheet = null
      //timesheet object is sorted by days so monday is 0, tuesday is 1, wednesday is 2, etc.
      this.timesheet = timesheet
      this.view = 'timesheet'
    },
    saveTasks() {
      const parsed = JSON.stringify(this.tasks);
      localStorage.setItem("tasks", parsed);
      this.pendingDelete = null;
      this.modalActive = false;
    },
    getTimeStamp(){
      let index = this.selectedDay
      var curr = new Date; // get current date
      var first = curr.getDate() - curr.getDay() + +index; // First day is the day of the month - the day of the week
      let [month, day] = new Date(curr.setDate(first)).toLocaleDateString("en-US").split("/");

      return `${month}/${day}`
    },
    getTotalTime(){
      let totalTime = 0
      this.tasks.map(task => {totalTime += task.elapsedTime})
      return this.secondsToHms(totalTime)

      // return date.toISOString().substr(11,8)
    },
    getDailyTime(index){
      var curr = new Date; // get current date
      var first = curr.getDate() - curr.getDay() + +index; // First day is the day of the month - the day of the week
      let [month, day] = new Date(curr.setDate(first)).toLocaleDateString("en-US").split("/");

      let totalTime = 0
      this.tasks.map(task => {
        if (task.date == `${month}/${day}`)
          totalTime += task.elapsedTime
        })
      let date = new Date(0)
      date.setSeconds(+totalTime)
      return `${month}/${day} - ${date.toISOString().substr(11,5)}`
    },

    getNewTaskIndex(){
      let lastTask = this.tasks[this.tasks.length - 1]
      if (!lastTask) {
        return 0
      } else {
        return +lastTask.id++
      }
      
    },


    secondsToHms(d) {
      d = Number(d);
      var h = Math.floor(d / 3600);
      var m = Math.floor(d % 3600 / 60);
      var s = Math.floor(d % 3600 % 60);
      console.log('mlength', m)
      if(m < 10){
        m = '0' + m
        console.log('ddy', m)
      }

      return `${h}:${m}:${s}`; 
    }
  },
  mounted() {

    if (localStorage.getItem("tasks")) {
      try {
        this.tasks = JSON.parse(localStorage.getItem("tasks"));
        this.totalTime = 0;
        this.tasks.forEach(task => {
          this.totalTime += task.elapsedTime;
        });
        const date = new Date(null);
        date.setSeconds(this.totalTime / 1000);
        const utc = date.toUTCString();
        this.totalTime = utc.substr(utc.indexOf(":") - 2, 8);
              [1,2,3,4,5,6,7].map(day => {
        this.dateCodes.push(this.getDailyTime(day).split(' - ')[0])
      })
      } catch (e) {
        localStorage.removeItem("tasks");
      }
    }
    // this.getDateTabs()
    this.getNewTaskIndex()
  },
  watch: {
    tasks: function(){
      let taskHeadings = []
      this.tasks.map(task => {
        taskHeadings.push(task.item)
      })
      this.taskHeadings = [...new Set(taskHeadings)]
    }
  }
};
</script>

<style>

@import url("https://fonts.googleapis.com/css2?family=Oxygen:wght@400;700&display=swap");
* {
  margin: 0px;
}

body {
  font-family: "Oxygen";
  background: #eeeeee;
  color: #222831;
}

.wrapper {
  padding: 2.5% 10%;
}
.wrapper .header {
  display: grid;
  grid-template-columns: 88% auto;
}
.c-button__control.export-tasks {
  background-image: url(/time-tracker/dist/img/fi-rr-disk.svg);
  background-size: 30px;
  background-repeat: no-repeat;
  background-position: center;
}

.c-button__control {
    color: white;
    font-size: 42px;
    font-weight: 400;
    background: lightgreen;
    height: 50px;
    width: 50px;
    border-radius: 50%;
    border: 2px solid #393e46;
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 2px 2px 5px 2px gainsboro;
    position: fixed;
    top: 20px;
}
.c-button__control.add-tasks {
  right: 80px;
}
.c-button__control.export-tasks {
  right: 20px;
}
.c-button__control.view-timesheet {
  right: 20px;
}

.c-button__control:hover {
  color: lightgreen;
  background: white;
}
.wrapper .header p {
  padding-left: 10px;
}

h1 {
  font-weight: 700;
  font-size: 48px;
  margin: 0px;
}

.task-list {
  width: 100%;
  padding-left: 0px;
  margin-left: 0px;
  margin-top: 40px;
}

.day-tabs {
  display: flex;
  list-style: none;
  background: white;
  padding: 20px;
  justify-content: space-between;
  border-radius: 12px;
  box-shadow: 2px 2px 5px 2px gainsboro;
  align-items: center;
}
.day-tab {
    background: #eeeeee;
    padding: 10px;
    color: black;
    text-decoration: none;
    font-weight: bold;
    border-radius: 10px;
    box-shadow: 2px 2px 5px 2px white;
    display: flex;
    flex-direction: column;
    text-decoration: none;
    cursor: pointer;
}
.day-tab:hover {
  background: lightsalmon;
}
.day-tab a {
  text-decoration: none;
  color: lightslategrey
}
.day-tab span {
  font-size: 75%;
}
.day-tab.selected {
  background: lightgreen;
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
  .task span {
    padding: 10px;
  }
  .task {
    grid-template-columns: auto auto auto auto;
  }
  .task input {
    padding-left: 10px;
  }
}
.copyright {
    position: fixed;
    bottom: 20px;
      left: 10%;
      right: 25%;
      z-index: -9;
}
.copyright a {
  font-weight: bold;
  text-decoration: none;
  color:#393e46;
}

@media print {
  .header, .copyright {
    display: none !important;
  }
  .wrapper {
    padding: 0 !important;
  }
}
</style>
