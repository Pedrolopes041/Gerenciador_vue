<template>
  <div class="min-h-screen bg-gray-100 py-12 px-4 sm:px-6 lg:px-8">
    <div class="max-w-3xl mx-auto">
      <header class="text-center mb-12">
        <h1 class="text-4xl font-extrabold text-gray-900 sm:text-5xl">
          Task Manager
        </h1>
      </header>

      <div class="mb-8">
        <button
          @click="toggleComponent()"
          class="w-full flex justify-center py-3 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition duration-150 ease-in-out"
        >
          <PlusIcon class="h-5 w-5 mr-2" />
          Add New Task
        </button>
      </div>

      <TransitionGroup
        name="task-list"
        tag="ul"
        class="space-y-4"
      >
        <li
          v-for="task in tasks"
          :key="task.id"
          class="bg-white shadow-md rounded-lg overflow-hidden transition duration-300 ease-in-out hover:shadow-lg"
        >
          <div class="p-6">
            <h3 class="text-lg font-semibold text-gray-900 mb-2">{{ task.name }}</h3>
            <p class="text-gray-600">{{ task.description }}</p>
          </div>
          <div class="bg-gray-50 px-6 py-4 flex justify-end space-x-3">
            <button
              @click="toggleComponent(task, false)"
              class="inline-flex items-center px-4 py-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition duration-150 ease-in-out"
            >
              <PencilIcon class="h-4 w-4 mr-2" />
              Update
            </button>
            <button
              @click="deleteTask(task.id)"
              class="inline-flex items-center px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-red-600 hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-150 ease-in-out"
            >
              <TrashIcon class="h-4 w-4 mr-2" />
              Delete
            </button>
          </div>
        </li>
      </TransitionGroup>

      <AddOrUpdate
        @close="handleClose"
        v-if="show"
        :data="data"
        :isNew="isNew"
        @task-updated="fetchTasks"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { db } from '../firebase.js'
import { getDocs, collection, deleteDoc, doc } from 'firebase/firestore'
import AddOrUpdate from './AddOrUpdate.vue'
import { PlusIcon, PencilIcon, TrashIcon } from 'lucide-vue-next'

const tasks = ref([])
const show = ref(false)
const data = ref({})
const isNew = ref(true)

onMounted(async () => {
  await fetchTasks()
})

async function fetchTasks() {
  tasks.value = []
  const tasksCollection = await getDocs(collection(db, 'tasks'))
  tasksCollection.forEach((task) => {
    tasks.value.push({ ...task.data(), id: task.id })
  })
}

async function deleteTask(taskId) {
  try {
    await deleteDoc(doc(db, 'tasks', taskId))
    await fetchTasks() // Refresh tasks list
  } catch (error) {
    console.error('Error deleting the document:', error)
  }
}

function toggleComponent(docData = {}, isNewDoc = true) {
  data.value = docData
  isNew.value = isNewDoc
  show.value = !show.value
}

function handleClose() {
  show.value = false
}
</script>

<style scoped>
.task-list-enter-active,
.task-list-leave-active {
  transition: all 0.5s ease;
}
.task-list-enter-from,
.task-list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
</style>