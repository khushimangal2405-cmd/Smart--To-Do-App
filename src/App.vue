<template>
  <div :class="['app', darkMode ? 'dark' : 'light']">
<div v-if="message" class="toast">
  {{ message }}
</div>
    <!-- Header -->
 <header>

  <div class="title-section">

    

    <div>

      <h1>Smart To-Do</h1>

      <p>
        Organize your daily life beautifully
      </p>

    </div>

  </div>

  <div class="header-right">

    <div class="clock">
      🕒 {{ currentTime }}
    </div>

    <button
      class="theme-btn"
      @click="toggleDarkMode"
    >
      {{ darkMode ? "☀ Light" : "🌙 Dark" }}
    </button>

  </div>

</header>

    <!-- Add Task -->
    <div class="add-task">
  <div class="input-group">
    <input
      v-model="newTask"
      placeholder="✍️ Enter your task..."
      @keyup.enter="addTask"
    />
  </div>

  <div class="input-group">
    <select v-model="priority">
      <option>🔥 High</option>
      <option>⭐ Medium</option>
      <option>🌿 Low</option>
    </select>
  </div>

  <div class="input-group">
    <input
      type="date"
      v-model="dueDate"
    />
  </div>

  <button class="add-btn" @click="addTask">
    ➕ Add Task
  </button>

</div>
    <!-- Search -->
    <div class="search-container">
  <input
    class="search"
    placeholder="Search your tasks..."
    v-model="search"
  />
</div>

    <!-- Filter -->
    <div class="filters">

      <button
        :class="{active:filter==='all'}"
        @click="filter='all'"
      >
        All
      </button>

      <button
        :class="{active:filter==='active'}"
        @click="filter='active'"
      >
        Active
      </button>

      <button
        :class="{active:filter==='completed'}"
        @click="filter='completed'"
      >
        Completed
      </button>

    </div>
<div class="stats">

  <div class="card">
    <h3>📋</h3>
    <h2>{{ tasks.length }}</h2>
    <p>Total Tasks</p>
  </div>

  <div class="card">
    <h3>✅</h3>
    <h2>{{ completedTasks }}</h2>
    <p>Completed</p>
  </div>

  <div class="card">
    <h3>⏳</h3>
    <h2>{{ remainingTasks }}</h2>
    <p>Pending</p>
  </div>

  <div class="card">
    <h3>🔥</h3>
    <h2>{{ completionPercentage }}%</h2>
    <p>Productivity</p>
  </div>

</div>
    <!-- Progress -->
    <div class="quote-box">
  💡 <b>Quote of the Day</b><br>
  {{ quote }}
</div>
    <div class="progress">

      <div class="progress-bar">
        <div
          class="progress-fill"
          :style="{width: completionPercentage + '%'}"
        ></div>
      </div>

      <p>{{ completionPercentage }}% Completed</p>

    </div>

    <!-- Task List -->
    <TransitionGroup name="list" tag="ul">

      <li
  v-for="task in filteredTasks"
  :key="task.id"
  class="task-card"
  :class="task.priority.toLowerCase().replace('🔥 ','').replace('⭐ ','').replace('🌿 ','')"
>

        <input
          type="checkbox"
          v-model="task.completed"
        />

        <div class="task-info">

          <span
            :class="{done:task.completed}"
          >
            {{ task.text }}
          </span>

          <small>
  Priority :
  <span
  :class="[
    'badge',
    task.priority.toLowerCase().replace('🔥 ','').replace('⭐ ','').replace('🌿 ','')
  ]"
>
  {{ task.priority }}
</span>
</small>

          <small
  :style="{
    color: !task.date
      ? 'gray'
      : new Date(task.date) < new Date()
      ? 'red'
      : '#43a047'
  }"
>
  Due:
  {{ task.date || "No Date" }}
  <span v-if="task.date && new Date(task.date) < new Date()">
    (Overdue)
  </span>
</small>
<div class="task-progress">

  <small>
    Progress: {{ task.progress || 0 }}%
  </small>

  <input
    type="range"
    min="0"
    max="100"
    v-model="task.progress"
  />

</div>
<div class="task-status">

  <small>Status:</small>

  <select v-model="task.status">

    <option>Not Started</option>

    <option>In Progress</option>

    <option>Completed</option>

  </select>

</div>
        </div>

        <button
          @click="editTask(task)"
        >
          ✏
        </button>

        <button
          @click="deleteTask(task.id)"
        >
          🗑
        </button>

      </li>

    </TransitionGroup>

    <!-- Counter -->
<div class="achievement-box">

  <h3>🏆 Achievement</h3>

  <p v-if="completionPercentage===100">
    🎉 Excellent! All Tasks Completed
  </p>

  <p v-else-if="completionPercentage>=75">
    🔥 Great Progress
  </p>

  <p v-else-if="completionPercentage>=50">
    👍 Keep Going
  </p>

  <p v-else>
    💪 Start Completing Tasks
  </p>

</div>
    <div class="task-counter">

      Total :
      {{ tasks.length }}

      |

      Completed :
      {{ completedTasks }}

      |

      Remaining :
      {{ remainingTasks }}

    </div>
<footer class="footer">

  <p>
    Made with ❤️ by <b>Khushi Mangal</b>
  </p>

  <p>
    Smart To-Do App • Version 1.0
  </p>

</footer>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from "vue"
const message = ref("")
const newTask = ref("")
const tasks = ref([])

const priority = ref("Medium")
const dueDate = ref("")

const search = ref("")
const filter = ref("all")

const darkMode = ref(false)

const currentTime = ref("")
const quotes = [
  "Success is the sum of small efforts repeated daily.",
  "Small progress is still progress.",
  "Discipline beats motivation.",
  "Stay focused and never give up.",
  "Dream big, work hard."
]

const quote = ref(
  quotes[Math.floor(Math.random() * quotes.length)]
)
const updateClock = () => {
  currentTime.value = new Date().toLocaleTimeString()
}

setInterval(updateClock,1000)

updateClock()

const toggleDarkMode = () => {
  darkMode.value=!darkMode.value
}

const addTask=()=>{

if(newTask.value.trim()==="") return

tasks.value.push({

id:Date.now(),

text:newTask.value,

completed:false,

priority:priority.value,

date:dueDate.value,

progress:0,

status:"Not Started"
})

newTask.value=""
message.value = "✅ Task Added Successfully"

setTimeout(() => {
  message.value = ""
}, 2000)
priority.value="Medium"

dueDate.value=""

}
const editTask = (task) => {
  const updated = prompt("Edit Task", task.text)

  if (updated !== null && updated.trim() !== "") {
    task.text = updated.trim()
  }
}

const deleteTask = (id) => {
  if (confirm("Are you sure you want to delete this task?")) {
    tasks.value = tasks.value.filter(task => task.id !== id)
  }
}

const filteredTasks = computed(() => {
  return tasks.value
  .filter(task => {

    const matchSearch = task.text
      .toLowerCase()
      .includes(search.value.toLowerCase())

    if (!matchSearch) return false

    if (filter.value === "completed") {
      return task.completed
    }

    if (filter.value === "active") {
      return !task.completed
    }

    return true
  })
  .sort((a, b) => {
    const order = {
  "🔥 High": 1,
  "⭐ Medium": 2,
  "🌿 Low": 3
}

    return order[a.priority] - order[b.priority]
  })
})

const completedTasks = computed(() => {
  return tasks.value.filter(task => task.completed).length
})

const remainingTasks = computed(() => {
  return tasks.value.length - completedTasks.value
})

const completionPercentage = computed(() => {

  if (tasks.value.length === 0) return 0

  return Math.round(
    (completedTasks.value / tasks.value.length) * 100
  )
})
watch(
  tasks,
  (newTasks)=>{
    newTasks.forEach(task=>{

      if(task.completed){
        task.status="Completed"
        task.progress=100
      }
      else if(task.progress>0){
        task.status="In Progress"
      }
      else{
        task.status="Not Started"
      }

    })
  },
  {deep:true}
)
watch(
  tasks,
  (newTasks) => {
    localStorage.setItem(
      "todoTasks",
      JSON.stringify(newTasks)
    )
  },
  { deep: true }
)
watch(
  tasks,
  (newTasks) => {
    localStorage.setItem(
      "todoTasks",
      JSON.stringify(newTasks)
    )
  },
  { deep: true }
)

watch(
  darkMode,
  (value) => {
    localStorage.setItem(
      "darkMode",
      JSON.stringify(value)
    )
  }
)

onMounted(() => {

  const savedTasks = localStorage.getItem("todoTasks")

  if (savedTasks) {

  tasks.value = JSON.parse(savedTasks).map(task => ({
    ...task,
    progress: task.progress || 0,
    status: task.status || "Not Started"
  }))

}

  const savedTheme = localStorage.getItem("darkMode")

  if (savedTheme) {
    darkMode.value = JSON.parse(savedTheme)
  }

})

</script>
<style scoped>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
  font-family:'Segoe UI',sans-serif;
}

body{
  min-height:100vh;
  background:linear-gradient(135deg,#0f172a,#1e293b,#2563eb);
  display:flex;
  justify-content:center;
  align-items:center;
  padding:25px;
}

.app{
  width:100%;
  max-width:1300px;
  padding:35px;
  border-radius:28px;
  background:rgba(255,255,255,.12);
  backdrop-filter:blur(22px);
  border:1px solid rgba(255,255,255,.15);
  box-shadow:0 20px 50px rgba(0,0,0,.35);
  transition:.4s;
}

.light{
  color:#222;
}

.dark{
  background:rgba(18,18,28,.92);
  color:white;
}

header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:35px;
  padding-bottom:25px;
  border-bottom:1px solid rgba(255,255,255,.15);
}

.title-section h1{
  font-size:48px;
  font-weight:900;
  color:#00bfff;
  margin:0;
  line-height:1.1;
  letter-spacing:1px;
  text-shadow:0 0 12px rgba(0,191,255,.45);
}
.title-section{
  display:flex;
  flex-direction:column;
  justify-content:center;
}

.title-section p{
  margin-top:8px;
  font-size:18px;
  color:#d1d5db;
}
header{
  overflow:visible;
}

.header-right{
  display:flex;
  align-items:center;
  gap:18px;
}

.clock{
  padding:12px 18px;
  border-radius:14px;
  background:rgba(255,255,255,.10);
  backdrop-filter:blur(10px);
  font-size:18px;
  font-weight:bold;
}

.theme-btn{
  border:none;
  padding:12px 20px;
  border-radius:14px;
  cursor:pointer;
  background:linear-gradient(135deg,#00c6ff,#0072ff);
  color:white;
  font-weight:bold;
}

.theme-btn:hover{
  transform:translateY(-3px);
}

.add-task{
  display:grid;
  grid-template-columns:2fr 1fr 1fr auto;
  gap:15px;
  margin:25px 0;
}

.add-task input,
.add-task select{
  padding:15px;
  border:none;
  border-radius:15px;
  outline:none;
  background:rgba(255,255,255,.12);
  color:white;
  font-size:15px;
}

.dark .add-task input,
.dark .add-task select{
  background:#2d3748;
}

.add-task button{
  border:none;
  border-radius:15px;
  background:linear-gradient(135deg,#00c6ff,#0072ff);
  color:white;
  font-weight:bold;
  cursor:pointer;
}

.add-task button:hover{
  transform:translateY(-3px);
  box-shadow:0 10px 20px rgba(0,114,255,.35);
}

.search{
  width:100%;
  margin-bottom:20px;
  padding:15px;
  border:none;
  border-radius:15px;
  outline:none;
  background:rgba(255,255,255,.12);
  color:white;
}

.dark .search{
  background:#2d3748;
}

::placeholder{
  color:#cfd8e3;
}
.filters{
  display:flex;
  gap:15px;
  margin:25px 0;
}

.filters button{
  flex:1;
  padding:14px;
  border:none;
  border-radius:15px;
  background:rgba(255,255,255,.10);
  color:white;
  font-weight:700;
  cursor:pointer;
  transition:.3s;
}

.filters button:hover{
  transform:translateY(-3px);
}

.filters button.active{
  background:linear-gradient(135deg,#00c6ff,#0072ff);
  box-shadow:0 10px 20px rgba(0,114,255,.35);
}

.stats{
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(220px,1fr));
  gap:20px;
  margin:30px 0;
}

.card{
  background:rgba(255,255,255,.10);
  backdrop-filter:blur(15px);
  border:1px solid rgba(255,255,255,.15);
  border-radius:20px;
  padding:25px;
  text-align:center;
  transition:.35s;
  box-shadow:0 10px 25px rgba(0,0,0,.18);
}

.dark .card{
  background:rgba(255,255,255,.06);
}

.card:hover{
  transform:translateY(-8px) scale(1.03);
  box-shadow:0 18px 35px rgba(0,198,255,.25);
}

.card h3{
  font-size:34px;
  margin-bottom:10px;
}

.card h2{
  font-size:32px;
  color:#00c6ff;
  margin-bottom:8px;
}

.card p{
  color:#cbd5e1;
  font-size:15px;
}

.progress{
  margin:30px 0;
}

.progress-bar{
  width:100%;
  height:18px;
  background:rgba(255,255,255,.15);
  border-radius:30px;
  overflow:hidden;
}

.progress-fill{
  height:100%;
  background:linear-gradient(90deg,#00e5ff,#00b0ff,#2979ff);
  border-radius:30px;
  transition:width .8s ease;
  box-shadow:0 0 15px #00d4ff;
}

.progress p{
  margin-top:12px;
  text-align:center;
  font-weight:bold;
  color:#ffffff;
}
ul{
  list-style:none;
  margin-top:25px;
}

.task-card{
  position:relative;
  display:flex;
  align-items:center;
  gap:18px;
  padding:22px;
  margin-bottom:18px;
  border-radius:20px;
  background:rgba(255,255,255,.10);
  backdrop-filter:blur(18px);
  border:1px solid rgba(255,255,255,.15);
  box-shadow:0 12px 25px rgba(0,0,0,.18);
  transition:.35s;
}

.dark .task-card{
  background:rgba(255,255,255,.06);
}

.task-card::before{
  content:"";
  position:absolute;
  left:0;
  top:0;
  width:6px;
  height:100%;
  border-radius:20px 0 0 20px;
  background:linear-gradient(#00c6ff,#0072ff);
}

.task-card:hover{
  transform:translateY(-6px);
  box-shadow:0 18px 35px rgba(0,198,255,.25);
}

.task-info{
  flex:1;
}

.task-info span{
  display:block;
  font-size:20px;
  font-weight:700;
  margin-bottom:8px;
}

.task-info small{
  display:block;
  margin-top:4px;
  opacity:.85;
}

.done{
  text-decoration:line-through;
  color:#9ca3af;
}

.badge{
  display:inline-block;
  padding:5px 12px;
  border-radius:20px;
  color:white;
  font-size:12px;
  font-weight:bold;
}

.high{
  background:#ef4444;
  box-shadow:0 0 12px rgba(239,68,68,.6);
}

.medium{
  background:#f59e0b;
  box-shadow:0 0 12px rgba(245,158,11,.6);
}

.low{
  background:#22c55e;
  box-shadow:0 0 12px rgba(34,197,94,.6);
}

input[type="checkbox"]{
  width:22px;
  height:22px;
  accent-color:#0072ff;
  cursor:pointer;
}

.task-card button{
  width:45px;
  height:45px;
  border:none;
  border-radius:50%;
  display:flex;
  justify-content:center;
  align-items:center;
  cursor:pointer;
  font-size:18px;
}

.task-card button:first-of-type{
  background:#22c55e;
  color:white;
}

.task-card button:last-of-type{
  background:#ef4444;
  color:white;
}

.task-counter{
  margin-top:25px;
  text-align:center;
  font-weight:bold;
}

.toast{
  position:fixed;
  top:25px;
  right:25px;
  background:#10b981;
  color:white;
  padding:15px 22px;
  border-radius:14px;
  box-shadow:0 12px 25px rgba(0,0,0,.25);
  animation:slideDown .4s;
  z-index:999;
}

@keyframes slideDown{
  from{
    opacity:0;
    transform:translateY(-40px);
  }
  to{
    opacity:1;
    transform:translateY(0);
  }
}

.list-enter-active,
.list-leave-active{
  transition:.4s;
}

.list-enter-from{
  opacity:0;
  transform:translateY(-20px);
}

.list-leave-to{
  opacity:0;
  transform:translateX(30px);
}

::-webkit-scrollbar{
  width:8px;
}

::-webkit-scrollbar-thumb{
  background:#0072ff;
  border-radius:20px;
}

@media(max-width:900px){

header{
  flex-direction:column;
  gap:20px;
}

.add-task{
  grid-template-columns:1fr;
}

.filters{
  flex-direction:column;
}

.stats{
  grid-template-columns:1fr;
}

.task-card{
  flex-direction:column;
  align-items:flex-start;
}

.task-card button{
  width:100%;
  border-radius:12px;
}

}
.task-card{
  overflow:hidden;
}

.task-card::after{
  content:"";
  position:absolute;
  top:0;
  left:-100%;
  width:100%;
  height:100%;
  background:linear-gradient(
    90deg,
    transparent,
    rgba(255,255,255,.15),
    transparent
  );
  transition:.6s;
}

.task-card:hover::after{
  left:100%;
}

.card{
  position:relative;
  overflow:hidden;
}

.card::after{
  content:"";
  position:absolute;
  top:-50%;
  left:-50%;
  width:200%;
  height:200%;
  background:radial-gradient(
    rgba(0,191,255,.12),
    transparent 70%
  );
  opacity:0;
  transition:.5s;
}

.card:hover::after{
  opacity:1;
}
.add-btn{
  font-size:16px;
  letter-spacing:.5px;
}

.add-btn:hover{
  transform:translateY(-4px) scale(1.03);
}
.search:focus{
  box-shadow:0 0 20px rgba(0,191,255,.45);
}
.quote-box{
  margin:25px 0;
  padding:18px;
  border-radius:18px;
  background:rgba(255,255,255,.12);
  backdrop-filter:blur(15px);
  border-left:6px solid #00c6ff;
  font-size:17px;
  line-height:1.7;
  box-shadow:0 10px 25px rgba(0,0,0,.2);
}

.dark .quote-box{
  background:rgba(255,255,255,.08);
}
.achievement-box{
  margin:30px 0;
  padding:22px;
  border-radius:20px;
  text-align:center;
  background:linear-gradient(135deg,#1d4ed8,#06b6d4);
  color:white;
  box-shadow:0 12px 30px rgba(0,0,0,.25);
}

.achievement-box h3{
  margin-bottom:10px;
  font-size:24px;
}

.achievement-box p{
  font-size:18px;
  font-weight:bold;
}
.footer{
  margin-top:40px;
  padding-top:20px;
  border-top:1px solid rgba(255,255,255,.15);
  text-align:center;
}

.footer p{
  margin:6px 0;
  color:#cbd5e1;
  font-size:15px;
}

.footer b{
  color:#00c6ff;
}
.add-task input:focus,
.add-task select:focus,
.search:focus{
  transform:scale(1.02);
}
button{
  transition:all .3s ease;
}

button:hover{
  transform:translateY(-3px);
}

button:active{
  transform:scale(.95);
}
.done{
  text-decoration:line-through;
  color:#9ca3af;
  opacity:.7;
  transition:.3s;
}
.card h2{
  animation:pop .5s;
}

@keyframes pop{
  0%{
    transform:scale(.8);
  }

  100%{
    transform:scale(1);
  }
}
.task-progress{
  margin-top:12px;
}

.task-progress input{
  width:100%;
  cursor:pointer;
}

.task-progress small{
  font-weight:bold;
}
.task-card.high::before{
  background:#ef4444;
}

.task-card.medium::before{
  background:#f59e0b;
}

.task-card.low::before{
  background:#22c55e;
}
.task-status{
  margin-top:10px;
}

.task-status select{
  padding:6px 10px;
  border-radius:10px;
  border:none;
  cursor:pointer;
}
</style>