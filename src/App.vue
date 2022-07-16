<template>
  <div class="main">
    <img :src="source" alt="vue3 image" class="logo">
    <h1>{{ title }}</h1>
    <h2>Nom de la tâche en cours : {{ taskName }}</h2>
    <h3>Tache en cours : {{ (isTaskInProgress) ? 'oui' : 'non' }}</h3>
    <h3>Durée en cours : {{ currentDuration }}</h3>
    
    <button @click="toggleTask" class="button" :class="isTaskInProgress ? 'red' : 'blue'">{{ isTaskInProgress ? 'Arrêter' : 'Démarer'}}</button>

    <div class="error">
      <p>{{ errorMsg }}</p>
    </div>

    <div>
      <input type="text" v-model="taskName">
    </div>

    <table>
      <thead>
        <tr>
          <th>id</th>
          <th>Tâche</th>
          <th>Début</th>
          <th>Fin</th>
          <th>Durée</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="task in tasks">
          <th>{{ task.id }}</th>
          <th>{{ task.name }}</th>
          <th>{{ formatTimestramp(task.start) }}</th>
          <th>{{ formatTimestramp(task.end) }}</th>
          <th>{{ durationBetweenTimestamps(task.start, task.end) }}</th>
        </tr>
      </tbody>
    </table>

  </div>

</template>

<script>
export default {
  data() {
    return {
      title: 'Hello World',
      source: 'https://cdn.svgporn.com/logos/vue.svg',
      tsFormatter: Intl.DateTimeFormat('fr', {hour: '2-digit', minute: '2-digit'}),
      tasks: [],
      taskID: 0,
      taskName: '',
      isTaskInProgress: false,
      startTime: null,
      nowTime: null,
      internalEverySecond: null,
      errorMsg: null
    }
  },
  computed: {
    currentDuration () {
      if (this.startTime && this.nowTime) {
        return this.durationBetweenTimestamps(this.startTime, this.nowTime)
      } else {
        return '00:00:00'
      }
      
    }
  },
  watch: {
    isTaskInProgress(isInProgress) {
      if (isInProgress) {
        this.internalEverySecond = setInterval(() => {
          this.nowTime = Date.now()
        }, 1000)
      } else {
        clearInterval(this.internalEverySecond)
      }
    }
  },
  methods: {
    startTask () {
      // Vérifications
      if (this.taskName.length == 0) {
        this.errorMsg = "Le nom d'une tâche ne peut pas être vide"
        return
      } else if (this.isTaskInProgress) {
        this.errorMsg = "une tâche est déjà en cours"
        return
      } else {
        this.errorMsg = null
      }

      // Début de la tâche
      this.isTaskInProgress = true
      this.startTime = Date.now()
      this.nowTime = Date.now()
    },
    stopTask () {
      // Vérifications
      if (!this.isTaskInProgress) {
        this.errorMsg = "Aucune tâche n'est en cours"
        return
      }

      // Enregistrement de la tâche
      this.tasks.unshift({
        id: this.getAnID(),
        name: this.taskName,
        start: this.startTime,
        end: Date.now()
      })

      // Fin de la tâche
      clearInterval(this.internalEverySecond)
      this.isTaskInProgress = false
      this.errorMsg = null
      this.nowTime = null
      this.taskName = ''
    },
    toggleTask() {
      if (this.isTaskInProgress) {
        this.stopTask()
      } else {
        this.startTask()
      }
    },
    getAnID() {
      this.taskID++
      return this.taskID
    },
    formatTimestramp(ts) {
      return this.tsFormatter.format(ts)
    },
    durationBetweenTimestamps (start, end) {
      console.log(start, end)
      let seconds = Math.floor((end / 1000) - (start / 1000))
      let minutes = Math.floor(seconds / 60)
      const hours = Math.floor(minutes / 60)
      seconds = seconds % 60
      minutes = minutes % 60
      return `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
    }
  },
}
</script>

<style scoped>
.main {
  text-align: center;
}
.error {
  background: red;
}

.red {
  background: red;
  color: white;
  border: none;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
}
.blue {
  background: lightblue;
  border: none;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
}
.button {
  margin: 5px;
}
</style>