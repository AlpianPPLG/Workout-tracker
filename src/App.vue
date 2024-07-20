<template>
  <div class="min-h-screen bg-gray-100 p-4">
    <header class="text-center py-4 mr-10">
      <h1 class="text-4xl font-extrabold text-gray-900">Workout Tracker</h1>
      <div class="absolute top-0 right-0 mt-11 mr-4">
        <button @click="openSettings" class="text-gray-600 hover:text-gray-900">
          <font-awesome-icon icon="cog" size="lg" />
        </button>
        <button @click="openUserAccount" class="text-gray-600 hover:text-gray-900 ml-4">
          <font-awesome-icon icon="user" size="lg" />
        </button>
      </div>
    </header>
    <main class="max-w-4xl mx-auto bg-white rounded-lg shadow-lg p-6">
      <section class="mb-8">
        <h2 class="text-2xl font-semibold mb-4">Add Workout</h2>
        <form @submit.prevent="addWorkout" class="flex flex-col space-y-4">
          <input
            v-model="newWorkout.name"
            type="text"
            placeholder="Workout Name"
            class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
          <input
            v-model="newWorkout.date"
            type="date"
            class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
          <input
            v-model="newWorkout.duration"
            type="number"
            placeholder="Duration (minutes)"
            class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
          <select
            v-model="newWorkout.category"
            class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
          >
            <option value="">Select Category</option>
            <option v-for="category in categories" :key="category" :value="category">
              {{ category }}
            </option>
          </select>
          <input
            v-model="newWorkout.reminder"
            type="datetime-local"
            class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
          <button
            type="submit"
            class="bg-blue-600 text-white p-3 rounded-lg hover:bg-blue-700 transition"
          >
            Add Workout
          </button>
        </form>
      </section>

      <section class="mb-8">
        <h2 class="text-2xl font-semibold mb-4">Workout List</h2>
        <div class="flex flex-col md:flex-row space-y-4 md:space-x-4 mb-4">
          <input
            v-model="filterDate"
            type="date"
            class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Search by name"
            class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
        </div>
        <ul class="space-y-4">
          <li
            v-for="(workout, index) in filteredWorkouts"
            :key="index"
            class="p-4 bg-gray-50 rounded-lg shadow-lg flex justify-between items-center"
          >
            <div v-if="editIndex !== index" class="flex-1">
              <h3 class="text-xl font-semibold">{{ workout.name }}</h3>
              <p class="text-gray-700">{{ workout.date }}</p>
              <p class="text-gray-700">{{ workout.duration }} minutes</p>
              <p class="text-gray-700">{{ workout.category }}</p>
              <p class="text-gray-700">
                Reminder:
                {{ workout.reminder ? new Date(workout.reminder).toLocaleString() : 'None' }}
              </p>
            </div>
            <div v-else class="flex-1 flex flex-col space-y-4">
              <input
                v-model="editWorkout.name"
                type="text"
                placeholder="Workout Name"
                class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
              />
              <input
                v-model="editWorkout.date"
                type="date"
                class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
              />
              <input
                v-model="editWorkout.duration"
                type="number"
                placeholder="Duration (minutes)"
                class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
              />
              <select
                v-model="editWorkout.category"
                class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
              >
                <option value="">Select Category</option>
                <option v-for="category in categories" :key="category" :value="category">
                  {{ category }}
                </option>
              </select>
              <input
                v-model="editWorkout.reminder"
                type="datetime-local"
                class="p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
              />
              <button
                @click="saveEdit(index)"
                class="bg-green-600 text-white p-3 rounded-lg hover:bg-green-700 transition"
              >
                Save
              </button>
              <button
                @click="cancelEdit"
                class="bg-red-600 text-white p-3 rounded-lg hover:bg-red-700 transition"
              >
                Cancel
              </button>
            </div>
            <div class="flex space-x-2">
              <button
                v-if="editIndex !== index"
                @click="editWorkoutDetails(index)"
                class="bg-yellow-600 text-white p-3 rounded-lg hover:bg-yellow-700 transition"
              >
                Edit
              </button>
              <button
                @click="removeWorkout(index)"
                class="bg-red-600 text-white p-3 rounded-lg hover:bg-red-700 transition"
              >
                Remove
              </button>
            </div>
          </li>
        </ul>
      </section>

      <section class="mb-8">
        <h2 class="text-2xl font-semibold mb-4">Statistics</h2>
        <div class="bg-gray-50 p-4 rounded-lg shadow-lg">
          <p class="text-xl font-semibold">Total Duration: {{ totalDuration }} minutes</p>
          <p class="text-xl font-semibold">Average Duration: {{ averageDuration }} minutes</p>
          <p class="text-xl font-semibold">Workouts This Month: {{ workoutsThisMonth }}</p>
        </div>
      </section>

      <section>
        <h2 class="text-2xl font-semibold mb-4">Workout Trends</h2>
        <div class="bg-gray-50 p-4 rounded-lg shadow-lg">
          <canvas id="workoutChart"></canvas>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

export default {
  data() {
    return {
      newWorkout: {
        name: '',
        date: '',
        duration: 0,
        category: '',
        reminder: ''
      },
      workouts: [],
      editWorkout: {
        name: '',
        date: '',
        duration: 0,
        category: '',
        reminder: ''
      },
      editIndex: -1,
      filterDate: '',
      searchQuery: '',
      categories: ['Cardio', 'Strength', 'Flexibility', 'Balance']
    }
  },
  computed: {
    filteredWorkouts() {
      return this.workouts.filter((workout) => {
        const matchesDate = !this.filterDate || workout.date === this.filterDate
        const matchesSearch = workout.name.toLowerCase().includes(this.searchQuery.toLowerCase())
        return matchesDate && matchesSearch
      })
    },
    totalDuration() {
      return this.workouts.reduce((total, workout) => total + workout.duration, 0)
    },
    averageDuration() {
      return this.workouts.length ? (this.totalDuration / this.workouts.length).toFixed(2) : 0
    },
    workoutsThisMonth() {
      const today = new Date()
      const startOfMonth = new Date(today.getFullYear(), today.getMonth(), 1)
      return this.workouts.filter((workout) => new Date(workout.date) >= startOfMonth).length
    }
  },
  methods: {
    addWorkout() {
      if (!this.newWorkout.name || !this.newWorkout.date || this.newWorkout.duration <= 0) {
        Swal.fire({
          icon: 'warning',
          title: 'Oops...',
          text: 'Please fill out all fields!'
        })
        return
      }
      this.workouts.push({ ...this.newWorkout })
      this.newWorkout = {
        name: '',
        date: '',
        duration: 0,
        category: '',
        reminder: ''
      }
      this.updateChart()
    },
    removeWorkout(index) {
      this.workouts.splice(index, 1)
      this.updateChart()
    },
    editWorkoutDetails(index) {
      this.editIndex = index
      this.editWorkout = { ...this.workouts[index] }
    },
    saveEdit(index) {
      if (!this.editWorkout.name || !this.editWorkout.date || this.editWorkout.duration <= 0) {
        Swal.fire({
          icon: 'warning',
          title: 'Oops...',
          text: 'Please fill out all fields!'
        })
        return
      }
      this.workouts.splice(index, 1, { ...this.editWorkout })
      this.editIndex = -1
      this.editWorkout = {
        name: '',
        date: '',
        duration: 0,
        category: '',
        reminder: ''
      }
      this.updateChart()
    },
    cancelEdit() {
      this.editIndex = -1
      this.editWorkout = {
        name: '',
        date: '',
        duration: 0,
        category: '',
        reminder: ''
      }
    },
    updateChart() {
      const ctx = document.getElementById('workoutChart').getContext('2d')
      const labels = this.workouts.map((workout) => new Date(workout.date).toLocaleDateString())
      const data = this.workouts.map((workout) => workout.duration)

      // Clear previous chart
      if (this.chartInstance) {
        this.chartInstance.destroy()
      }

      // Create new chart
      this.chartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [
            {
              label: 'Workout Duration (minutes)',
              data: data,
              borderColor: 'rgba(75, 192, 192, 1)',
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              borderWidth: 1
            }
          ]
        },
        options: {
          responsive: true,
          scales: {
            x: {
              beginAtZero: true,
              ticks: {
                autoSkip: true,
                maxTicksLimit: 10
              }
            },
            y: {
              beginAtZero: true
            }
          }
        }
      })
    }
  },
  watch: {
    workouts() {
      this.updateChart()
    }
  },
  mounted() {
    this.updateChart()
  }
}
</script>
