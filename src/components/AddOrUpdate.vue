<script setup>
import { ref, onMounted, watch } from "vue";
import { db } from "../firebase.js";
import { collection, addDoc, doc, updateDoc } from "firebase/firestore";
import { XIcon, PlusIcon, PencilIcon } from 'lucide-vue-next';

const props = defineProps({
  data: Object,
  isNew: Boolean,
});

const emits = defineEmits(["close", "task-updated"]);

const task = ref({
  name: "",
  description: "",
});

onMounted(() => {
  if (props.data) {
    task.value = { ...props.data };
  }
});

watch(() => props.data, (newData) => {
  if (newData) {
    task.value = { ...newData };
  }
}, { deep: true });

async function addOrUpdate() {
  try {
    if (props.isNew) {
      await addDoc(collection(db, 'tasks'), task.value);
    } else {
      await updateDoc(doc(db, "tasks", props.data.id), task.value);
    }
    emits("task-updated");
    emits("close");
  } catch (error) {
    console.error("Erro ao adicionar ou atualizar o documento:", error);
  }
}
</script>

<template>
  <transition name="modal" appear>
    <div class="fixed inset-0 z-50 overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
      <div class="flex items-end justify-center min-h-screen px-4 pt-4 pb-20 text-center sm:block sm:p-0">
        <div class="fixed inset-0 transition-opacity bg-gray-500 bg-opacity-75" aria-hidden="true" @click="emits('close')"></div>

        <span class="hidden sm:inline-block sm:align-middle sm:h-screen" aria-hidden="true">&#8203;</span>

        <div class="inline-block overflow-hidden text-left align-bottom transition-all transform bg-white rounded-lg shadow-xl sm:my-8 sm:align-middle sm:max-w-lg sm:w-full">
          <div class="px-4 pt-5 pb-4 bg-white sm:p-6 sm:pb-4">
            <div class="sm:flex sm:items-start">
              <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left w-full">
                <h3 class="text-lg font-medium leading-6 text-gray-900" id="modal-title">
                  {{ props.isNew ? "Adicionar Nova Tarefa" : "Editar Tarefa" }}
                </h3>
                <div class="mt-2">
                  <input 
                    type="text" 
                    placeholder="Nome da Tarefa" 
                    v-model="task.name"
                    class="w-full px-3 py-2 text-gray-700 border rounded-lg focus:outline-none focus:border-blue-500" 
                  />
                  <textarea 
                    placeholder="Descrição da Tarefa" 
                    v-model="task.description"
                    class="w-full px-3 py-2 mt-4 text-gray-700 border rounded-lg focus:outline-none focus:border-blue-500" 
                    rows="4"
                  ></textarea>
                </div>
              </div>
            </div>
          </div>
          <div class="px-4 py-3 bg-gray-50 sm:px-6 sm:flex sm:flex-row-reverse">
            <button 
              @click="addOrUpdate" 
              class="inline-flex justify-center w-full px-4 py-2 text-base font-medium text-white bg-blue-600 border border-transparent rounded-md shadow-sm hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:ml-3 sm:w-auto sm:text-sm"
            >
              <PlusIcon v-if="props.isNew" class="w-5 h-5 mr-2" />
              <PencilIcon v-else class="w-5 h-5 mr-2" />
              {{ props.isNew ? "Adicionar" : "Atualizar" }}
            </button>
            <button 
              @click="emits('close')" 
              class="inline-flex justify-center w-full px-4 py-2 mt-3 text-base font-medium text-gray-700 bg-white border border-gray-300 rounded-md shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm"
            >
              <XIcon class="w-5 h-5 mr-2" />
              Fechar
            </button>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>