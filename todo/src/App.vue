<template>
  <div id="app">
    <h1 v-text="title"></h1>
    <p v-if="isEmpty">No tasks free woohoo~</p>
    <p v-if="!isEmpty">Currently finished <span class="progress">{{ progress + '%'}}</span> </p>
    <ul class="list" >
      <div class="list-input">
        <input v-model="newItem" @keyup.enter="addNew" placeholder="what are you up to">
      </div>
      <li class="list-item" v-for="(item, index) in list">
        <span :class="[{ finished: item.isFinished }, 'item-text']" @click="toggleFinish(item)">{{ item.label }}</span>
        <div class="item-due">
          <span class="hint" v-if="!item.dueDate" @click="loadDateField(item)">Add a due date</span>
          <input class="date-input" v-if="item.dueDate==='input'" v-model="newDueDate" @keyup.enter="addDue(item)" type="date">
          <span v-if="item.dueDate && item.dueDate!=='input'" >Due in {{ item.daysLeft }} days  </span>
          <span v-if="item.dueDate && item.dueDate!=='input'" class="hint hint-edit" @click="editDue(item)">edit due date</span>
        </div>
        <button class="btn delete-btn" v-on:click="list.splice(index, 1)">X</button>
      </li>
    </ul>
    <br>
    <button class="btn reset-btn" v-on:click="resetList">Clear Todos</button>
    <!-- <router-view></router-view> -->
    <footer>
      <span>LYCworks&copy;2016</span>
    </footer>
  </div>
</template>

<script>
import Store from './store' // getting default exported methods
import moment from 'moment'

const STORAGE_KEY = 'todo'

export default {
  name: 'app',
  data: () => {
    return {
      title: 'TODO',
      list: Store.fetch(STORAGE_KEY) || [],
      newItem: '',
      newDueDate: '',
      progress: 0,
      isEmpty: false,
      showDateField: false
    }
  },
  created: function () {
    this.updateProgress()
    this.updateCurrentDate()
  },
  methods: {
    isEmpty () {
      console.log(this.list.length)
      return this.list.length === 0
    },
    updateCurrentDate () {
      this.list = this.list.map((item) => {
        if (item.dueDate) {
          console.log('what', item.dueDate)
          item.daysLeft = moment(item.dueDate, 'YYYY-MM-DD').diff(moment().now, 'days') // 1
        }
        return item
      })
    },
    toggleFinish (item) {
      item.isFinished = !item.isFinished
    },
    loadDateField (item) {
      console.log('at leaast im here')
      item.dueDate = 'input'
    },
    addNew () {
      this.list.push({
        label: this.newItem,
        isFinished: false,
        dueDate: '',
        daysLeft: ''
      })
      this.newItem = ''
      Store.save(this.list)
    },
    addDue (item) {
      item.dueDate = this.newDueDate
      item.daysLeft = moment(this.newDueDate, 'YYYY-MM-DD').diff(moment().now, 'days') // 1
      // console.log('add', moment(this.dueDate, 'YYYY-MM-DD').diff(moment().now, 'days'))
    },
    editDue (item) {
      item.dueDate = 'input'
    },
    updateProgress () {
      if (this.list.length === 0) {
        this.progress = 0
      } else {
        this.progress = Math.floor(this.list.filter((item) => (item.isFinished)).length / this.list.length * 100)
      }
    },
    resetList () {
      this.list = []
    }
  },
  watch: { // value change
    list: {
      handler: function (val, oldVal) {
        Store.save(STORAGE_KEY, this.list)
        this.updateProgress()
        console.log('i detect the change')
      },
      deep: true // remembers inner attribute
    }
  }
}
</script>

<style>

.hint {
  color: #41b883;
}

.hint:hover {
  cursor: pointer;
}

.finished {
  text-decoration: line-through;
}

.progress {
  color: #41b883;
  font-weight: bold;
}

.list {
  list-style: none;
  padding: 0;
  margin: 0 auto;
  width: 50%;
}

.list-input {
  border-bottom: 1px solid #41b883;
  margin-bottom: 1em;
}

.list-input > input {
  width: 100%;
  border: none;
  padding: 0.5em;
  font-size: medium;
}

.date-input {
  width: 100%;
  border: none;
  font-size: medium;
  font-family: inherit;
  color: inherit;
}

.list-item {
  display: flex;
  width: 100%;
  padding: 0.5em;
  border-bottom: solid 1px #eee;
  align-items: center;
}

.list-item > span {
  text-align: left;
  margin-left: 1em;
}

.item-text {
  width: 80%;
}

.item-due {
  width: 50%;
  text-align: left;
  display: flex;
  width: 50%;
  justify-content: space-between;
}

.hint-edit {
  margin-left: 5px;
}

.archive {
  visibility: hidden;
}

.btn {
  border-radius: 5px;
  font-size: medium;
}

.btn:hover {
  cursor: pointer;
}

.reset-btn {
  padding: 1em;
  height: auto;
  color: white;
  background: #2C3E50;
  border: 2px solid #2C3E50;
}

.reset-btn:hover {
  background: white;
  color: #2C3E50;
}

.delete-btn {
  margin-left: 20px;
  width: 50px;
  height: auto;
  border: 2px solid #E91E63;
  background: white;
  color: #E91E63;
  font-weight: 800;
}

.delete-btn:hover {
  color: white;
  background: #E91E63;
}

footer {
  position: absolute;
  bottom: 20px;
  left: 45%;
  padding: 0;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
