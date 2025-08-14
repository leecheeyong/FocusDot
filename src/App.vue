<template>
  <div 
    :class="[
      'min-h-screen transition-all duration-500 relative overflow-hidden',
      isDark ? 'bg-gray-900' : 'bg-gray-50'
    ]"
    @keydown="handleKeyPress"
    tabindex="0"
  >
    <div class="absolute inset-0 pointer-events-none">
      <div 
        v-for="i in 6" 
        :key="i"
        :class="[
          'absolute rounded-full opacity-20 animate-pulse',
          isDark ? 'bg-blue-400' : 'bg-blue-200'
        ]"
        :style="{
          width: Math.random() * 100 + 50 + 'px',
          height: Math.random() * 100 + 50 + 'px',
          left: Math.random() * 100 + '%',
          top: Math.random() * 100 + '%',
          animationDelay: Math.random() * 3 + 's',
          animationDuration: (Math.random() * 4 + 3) + 's'
        }"
      />
    </div>

    <header :class="[
      'border-b px-6 py-4 backdrop-blur-sm relative z-10',
      isDark ? 'bg-gray-800/80 border-gray-700' : 'bg-white/80 border-gray-200'
    ]">
      <div class="max-w-4xl mx-auto flex items-center justify-between">
        <div class="flex items-center gap-4">
          <h1 :class="[
            'text-2xl font-bold',
            isDark ? 'text-white' : 'text-gray-900'
          ]">
            FocusDot
          </h1>
          <div v-if="totalTasks > 0" class="relative w-8 h-8">
            <svg class="w-8 h-8 transform -rotate-90" viewBox="0 0 32 32">
              <circle
                cx="16" cy="16" r="14"
                :class="isDark ? 'stroke-gray-700' : 'stroke-gray-200'"
                stroke-width="4"
                fill="none"
              />
              <circle
                cx="16" cy="16" r="14"
                :class="isDark ? 'stroke-blue-400' : 'stroke-blue-500'"
                stroke-width="4"
                fill="none"
                :stroke-dasharray="87.96"
                :stroke-dashoffset="87.96 - (87.96 * completedTasks.length / totalTasks)"
                class="transition-all duration-500"
              />
            </svg>
          </div>
        </div>
        
        <div class="flex items-center gap-4">
          <div :class="[
            'text-sm',
            isDark ? 'text-gray-300' : 'text-gray-500'
          ]">
            {{ completedTasks.length }} completed • {{ pendingTasks.length }} remaining
          </div>
          <button
            @click="toggleDarkMode"
            :class="[
              'p-2 rounded-lg transition-all duration-200 hover:scale-110',
              isDark ? 'bg-gray-700 text-yellow-400 hover:bg-gray-600' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
            ]"
            title="Toggle theme"
          >
            {{ isDark ? '☀️' : '🌙' }}
          </button>
        </div>
      </div>
    </header>

    <main class="flex-1 flex items-center justify-center px-6 py-12 relative z-10">
      <div class="w-full max-w-2xl">
        <div v-if="currentTask" class="text-center mb-12">
          <div class="mb-8 transform transition-all duration-500 hover:scale-105">
            <p :class="[
              'text-4xl md:text-5xl font-bold leading-tight mb-4 transition-colors duration-300',
              isDark ? 'text-white' : 'text-gray-900'
            ]">
              {{ currentTask.text }}
            </p>
            <p :class="[
              'text-lg',
              isDark ? 'text-gray-400' : 'text-gray-500'
            ]">
              Task {{ currentTaskIndex + 1 }} of {{ totalTasks }}
            </p>
          </div>
          
          <div class="flex flex-col sm:flex-row gap-4 justify-center">
            <button
              @click="completeCurrentTask"
              class="bg-green-500 hover:bg-green-600 text-white font-semibold py-4 px-8 rounded-xl transition-all duration-200 text-lg transform hover:scale-105 hover:shadow-lg"
            >
              ✓ Mark as Done
            </button>
            
            <button
              v-if="pendingTasks.length > 1"
              @click="skipToNext"
              :class="[
                'font-semibold py-4 px-8 rounded-xl transition-all duration-200 text-lg transform hover:scale-105',
                isDark ? 'bg-gray-700 hover:bg-gray-600 text-gray-200' : 'bg-gray-200 hover:bg-gray-300 text-gray-700'
              ]"
            >
              Skip for Now
            </button>
          </div>
        </div>

        <div v-else-if="tasks.length === 0" class="text-center">
          <div class="mb-8">
            <div :class="[
              'w-24 h-24 rounded-full flex items-center justify-center mx-auto mb-6 transition-all duration-300 hover:scale-110',
              isDark ? 'bg-gray-700' : 'bg-gray-200'
            ]">
              <span class="text-4xl animate-bounce">📝</span>
            </div>
            <h2 :class="[
              'text-3xl font-bold mb-4',
              isDark ? 'text-white' : 'text-gray-900'
            ]">Ready to Focus?</h2>
            <p :class="[
              'text-xl',
              isDark ? 'text-gray-400' : 'text-gray-600'
            ]">Add your first task to get started</p>
          </div>
        </div>

        <div v-else class="text-center">
          <div class="mb-8">
            <div class="w-24 h-24 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-6 animate-pulse">
              <span class="text-4xl animate-bounce">🎉</span>
            </div>
            <h2 :class="[
              'text-3xl font-bold mb-4',
              isDark ? 'text-white' : 'text-gray-900'
            ]">All Done!</h2>
            <p :class="[
              'text-xl mb-6',
              isDark ? 'text-gray-400' : 'text-gray-600'
            ]">You've completed all your tasks</p>
            <button
              @click="resetAllTasks"
              class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-3 px-6 rounded-xl transition-all duration-200 transform hover:scale-105"
            >
              Start Over
            </button>
          </div>
        </div>

        <div :class="[
          'rounded-xl shadow-lg border backdrop-blur-sm p-6 transition-all duration-300 hover:shadow-xl',
          isDark ? 'bg-gray-800/80 border-gray-700' : 'bg-white/80 border-gray-200'
        ]">
          <form @submit.prevent="addTask" class="flex flex-col sm:flex-row gap-4">
            <input
              v-model="newTaskText"
              type="text"
              placeholder="What do you need to focus on?"
              :class="[
                'flex-1 px-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none text-lg transition-all duration-200',
                isDark ? 'bg-gray-700 border-gray-600 text-white placeholder-gray-400' : 'bg-white border-gray-300 text-gray-900'
              ]"
              required
            >
            <button
              type="submit"
              class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-3 px-6 rounded-xl transition-all duration-200 whitespace-nowrap transform hover:scale-105"
            >
              Add Task
            </button>
          </form>
        </div>

        <div v-if="tasks.length > 0" :class="[
          'mt-8 rounded-xl shadow-lg border backdrop-blur-sm p-6 transition-all duration-300',
          isDark ? 'bg-gray-800/80 border-gray-700' : 'bg-white/80 border-gray-200'
        ]">
          <div class="flex items-center justify-between mb-4">
            <h3 :class="[
              'text-lg font-semibold',
              isDark ? 'text-white' : 'text-gray-900'
            ]">All Tasks</h3>
            <button
              @click="showAllTasks = !showAllTasks"
              class="text-blue-500 hover:text-blue-600 font-medium transition-colors duration-200"
            >
              {{ showAllTasks ? 'Hide' : 'Show' }} ({{ tasks.length }})
            </button>
          </div>
          
          <div v-if="showAllTasks" class="space-y-3">
            <div
              v-for="(task, index) in tasks"
              :key="task.id"
              :class="[
                'flex items-center justify-between p-3 rounded-xl transition-all duration-200 hover:scale-[1.02] group',
                task.completed 
                  ? (isDark ? 'bg-green-900/50 text-green-300' : 'bg-green-50 text-green-800')
                  : (isDark ? 'bg-gray-700/50' : 'bg-gray-50')
              ]"
            >
              <div class="flex items-center gap-3">
                <span
                  :class="[
                    'w-6 h-6 rounded-full flex items-center justify-center text-sm font-medium transition-all duration-200',
                    task.completed 
                      ? 'bg-green-500 text-white' 
                      : (isDark ? 'bg-gray-600 text-gray-300' : 'bg-gray-200 text-gray-700')
                  ]"
                >
                  {{ task.completed ? '✓' : index + 1 }}
                </span>
                <span 
                  :class="[
                    'transition-all duration-200',
                    task.completed ? 'line-through opacity-75' : '',
                    isDark ? 'text-gray-200' : 'text-gray-900'
                  ]"
                >
                  {{ task.text }}
                </span>
              </div>
              
              <button
                @click="removeTask(task.id)"
                :class="[
                  'opacity-0 group-hover:opacity-100 text-red-500 hover:text-red-600 p-1 rounded transition-all duration-200 hover:bg-red-50',
                  isDark ? 'hover:bg-red-900/20' : 'hover:bg-red-50'
                ]"
                title="Remove task"
              >
                🗑️
              </button>
            </div>
          </div>
        </div>
      </div>
    </main>

    <div v-if="showCelebration" class="fixed inset-0 pointer-events-none z-50">
      <div 
        v-for="i in 20" 
        :key="i"
        class="absolute animate-ping"
        :style="{
          left: Math.random() * 100 + '%',
          top: Math.random() * 100 + '%',
          animationDelay: Math.random() * 2 + 's'
        }"
      >
        🎉
      </div>
    </div>

    <div 
      v-if="showEasterEgg" 
      :class="[
        'fixed top-4 right-4 p-4 rounded-xl shadow-lg z-50 animate-bounce',
        isDark ? 'bg-purple-900 text-purple-200' : 'bg-purple-100 text-purple-800'
      ]"
    >
      Konami Code activated! You're a true focus master!
    </div>
    <footer class="py-8 text-center text-gray-500 text-sm">
      <p>
        Made with <span class="text-red-500">❤️</span> by
        <a
          href="https://github.com/leecheeyong"
          target="_blank"
          class="text-gray-700 hover:underline"
        >
          Chee Yong Lee
        </a>
      </p>
      <p class="mt-1">
        Project available as open source under the terms of
        <a
          href="https://github.com/leecheeyong/focusdot/blob/main/LICENSE"
          target="_blank"
          class="text-gray-700 hover:underline"
          >MIT License</a
        >
      </p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const isDark = ref(false)
const tasks = ref([])
const newTaskText = ref('')
const showAllTasks = ref(false)
const showCelebration = ref(false)
const showEasterEgg = ref(false)
const konamiCode = ref([])
const konamiSequence = ['ArrowUp', 'ArrowUp', 'ArrowDown', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'ArrowLeft', 'ArrowRight', 'KeyB', 'KeyA']

const pendingTasks = computed(() => tasks.value.filter(task => !task.completed))
const completedTasks = computed(() => tasks.value.filter(task => task.completed))
const currentTask = computed(() => pendingTasks.value[0] || null)
const currentTaskIndex = computed(() => {
  if (!currentTask.value) return -1
  return tasks.value.findIndex(task => task.id === currentTask.value.id)
})
const totalTasks = computed(() => tasks.value.length)

const toggleDarkMode = () => {
  isDark.value = !isDark.value
  localStorage.setItem('focusdot-dark-mode', JSON.stringify(isDark.value))
}

const loadDarkMode = () => {
  const stored = localStorage.getItem('focusdot-dark-mode')
  if (stored) {
    isDark.value = JSON.parse(stored)
  }
}

const saveToStorage = () => {
  localStorage.setItem('focusdot-tasks', JSON.stringify(tasks.value))
}

const loadFromStorage = () => {
  const stored = localStorage.getItem('focusdot-tasks')
  if (stored) {
    tasks.value = JSON.parse(stored)
  }
}

const addTask = () => {
  if (newTaskText.value.trim()) {
    const newTask = {
      id: Date.now() + Math.random(),
      text: newTaskText.value.trim(),
      completed: false,
      createdAt: new Date().toISOString()
    }
    tasks.value.push(newTask)
    newTaskText.value = ''
  }
}

const completeCurrentTask = () => {
  if (currentTask.value) {
    const taskIndex = tasks.value.findIndex(task => task.id === currentTask.value.id)
    if (taskIndex !== -1) {
      tasks.value[taskIndex].completed = true
      tasks.value[taskIndex].completedAt = new Date().toISOString()
      
      showCelebration.value = true
      setTimeout(() => {
        showCelebration.value = false
      }, 2000)
    }
  }
}

const skipToNext = () => {
  if (pendingTasks.value.length > 1) {
    const current = currentTask.value
    const currentIndex = tasks.value.findIndex(task => task.id === current.id)
    tasks.value.splice(currentIndex, 1)
    
    const lastPendingIndex = tasks.value.map(task => task.completed).lastIndexOf(false)
    tasks.value.splice(lastPendingIndex + 1, 0, current)
  }
}

const removeTask = (taskId) => {
  tasks.value = tasks.value.filter(task => task.id !== taskId)
}

const resetAllTasks = () => {
  tasks.value = tasks.value.map(task => ({
    ...task,
    completed: false,
    completedAt: null
  }))
}

const handleKeyPress = (event) => {
  konamiCode.value.push(event.code)
  if (konamiCode.value.length > konamiSequence.length) {
    konamiCode.value.shift()
  }
  
  if (konamiCode.value.length === konamiSequence.length && 
      konamiCode.value.every((key, index) => key === konamiSequence[index])) {
    showEasterEgg.value = true
    setTimeout(() => {
      showEasterEgg.value = false
    }, 3000)
    konamiCode.value = []
  }
}

onMounted(() => {
  loadFromStorage()
  loadDarkMode()
})

watch(tasks, saveToStorage, { deep: true })
</script>