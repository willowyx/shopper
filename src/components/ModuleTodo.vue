<script setup lang="js">
import { onBeforeUnmount, ref, watch } from 'vue'
import ModuleFinalGraph from './ModulePresentation.vue'
import { BTable, BFormCheckbox, BCard } from 'bootstrap-vue-next'
import { version } from '../../package.json'

let dfid = 0

function createDefaultTodos() {
  return [
    { id: dfid++, text: 'Paper plates', qty: 1, img: 'noitem.png', done: true },
    { id: dfid++, text: 'Eggs', qty: 12, img: 'noitem.png', done: false },
  ]
}

const sgImageShow = ref(true)

watch(sgImageShow, (newValue) => {
  if (!newValue) {
    clearImage()
  }
})

const newTodo = ref('')
const newQty = ref(1)
const newImg = ref('')
const imageLockState = ref(false)
const fileInput = ref(null)
const importInput = ref(null)
const nameInput = ref('')
const todos = ref(createDefaultTodos())
const csname = ref('Alyssa')
const isInit = ref(false)
const showFinalGraph = ref(false)
let previousBodyOverflow = ''

watch(isInit, (newValue) => {
  if (newValue) {
    for (let i = 0; i < document.getElementsByClassName('todo-bc-wrapper').length; i++) {
      document.getElementsByClassName('todo-bc-wrapper')[i].classList.remove('novis')
    }
    for (let i = 0; i < document.getElementsByClassName('fc-home').length; i++) {
      document.getElementsByClassName('fc-home')[i].classList.add('novis')
    }
  } else {
    for (let i = 0; i < document.getElementsByClassName('todo-bc-wrapper').length; i++) {
      document.getElementsByClassName('todo-bc-wrapper')[i].classList.add('novis')
    }
    for (let i = 0; i < document.getElementsByClassName('fc-home').length; i++) {
      document.getElementsByClassName('fc-home')[i].classList.remove('novis')
    }
  }
})

watch(showFinalGraph, (isOpen) => {
  setBodyScrollLock(isOpen)
})

function startNew(list, name, isImport) {
  if (Array.isArray(list)) {
    todos.value = list
  } else {
    todos.value = createDefaultTodos()
  }
  if (name) {
    csname.value = name
  } else if (isImport) {
    csname.value = '' // fill name from import
  }
  isInit.value = true
  showFinalGraph.value = false
  clearImportSelection()
}

function startExport() {
  const exportData = {
    dataName: 'shopper-data',
    gameVersion: version,
    customerName: csname.value,
    data: todos.value,
  }
  const blob = new Blob([JSON.stringify(exportData, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `${csname.value}.json`
  a.click()
  URL.revokeObjectURL(url)
}

function deleteList() {
  if (!confirm("Clear all list data and return home?")) {
    return
  }

  todos.value = []
  isInit.value = false
  showFinalGraph.value = false
}

function clearImportSelection() {
  if (importInput.value) {
    importInput.value.value = ''
  }
}

function getImportedName(parsedData, fileName) {
  if (parsedData && typeof parsedData.customerName === 'string' && parsedData.customerName.trim()) {
    return parsedData.customerName.trim()
  }
  if (fileName.endsWith('.json')) {
    return fileName.slice(0, -5)
  }
  return ''
}

function normalizeImportedTodos(parsedData) {
  const importedTodos = Array.isArray(parsedData) ? parsedData : parsedData?.data
  if (!Array.isArray(importedTodos)) {
    return null
  }

  const normalized = importedTodos
    .filter((item) => item && typeof item === 'object')
    .map((item, index) => ({
      id: Number.isInteger(item.id) ? item.id : index,
      text: typeof item.text === 'string' ? item.text : '',
      qty: item.qty ?? 1,
      img: typeof item.img === 'string' ? item.img : 'noitem.png',
      done: Boolean(item.done),
    }))
    .filter((item) => item.text.trim().length > 0)

  return normalized
}

async function startImport() {
  const file = importInput.value?.files?.[0]
  if (!file) {
    alert('No file chosen')
    return
  }
  clearImportSelection()

  try {
    const fileText = await file.text()
    const parsedData = JSON.parse(fileText)
    const importedTodos = normalizeImportedTodos(parsedData)

    if (!importedTodos || importedTodos.length === 0) {
      alert('Invalid file')
      return
    }

    dfid = importedTodos.reduce((maxId, todo) => Math.max(maxId, todo.id), -1) + 1
    startNew(importedTodos, getImportedName(parsedData, file.name), true)
  } catch {
    alert('Unable to import file')
  }
}

function addTodo() {
  if (newImg.value) {
    if (!newImg.value.startsWith('https://') && !newImg.value.startsWith('data:image/')) {
      alert('Image URL must start with https:// or data:image/')
      return
    }
  }

  todos.value.push({
    id: dfid++,
    text: newTodo.value,
    qty: newQty.value,
    done: false,
    img: sgImageShow.value ? newImg.value || 'noitem.png' : '',
  })
  newTodo.value = ''
  newQty.value = 1
  newImg.value = ''
  imageLockState.value = false
  if (fileInput.value) {
    fileInput.value.value = ''
  }
}

function onFileChange(e) {
  const file = e.target.files[0]
  if (file) {
    if (!file.type.startsWith('image/')) {
      alert('File must be type: Image')
      if (fileInput.value) fileInput.value.value = ''
      return
    }
    const reader = new FileReader()
    reader.onload = (e) => {
      newImg.value = e.target.result
      imageLockState.value = true
    }
    reader.readAsDataURL(file)
  }
}

function clearImage() {
  newImg.value = ''
  imageLockState.value = false
  if (fileInput.value) {
    fileInput.value.value = ''
  }
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t.id !== todo.id)
}

function toggleFinalGraph() {
  showFinalGraph.value = !showFinalGraph.value
}

function toggleTodoFromPresentation({ todoId, isDone }) {
  const targetTodo = todos.value.find((todo) => todo.id === todoId)
  if (targetTodo) {
    targetTodo.done = Boolean(isDone)
  }
}

function setBodyScrollLock(isLocked) {
  if (isLocked) {
    previousBodyOverflow = document.body.style.overflow
    document.body.style.overflow = 'hidden'
    return
  }

  document.body.style.overflow = previousBodyOverflow
  previousBodyOverflow = ''
}

onBeforeUnmount(() => {
  setBodyScrollLock(false)
})

const fields = [
  {
    key: 'done',
    label: 'Complete',
    sortable: false, // todo: fix this
  },
  {
    key: 'text',
    label: 'Item',
    sortable: true,
  },
  {
    key: 'qty',
    label: 'Qty',
    sortable: true,
  },
  {
    key: 'img',
    label: 'Photo',
    sortable: false,
  },
  {
    key: 'delete',
    label: 'Manage',
    sortable: false,
  },
]
</script>

<template>
  <div class="fc-home">
    <BCard>
      Start a new shopping list
      <div class="formgroup fg-styled">
        <label class="formitem">
          Shopping for:
          <input type="text" required placeholder="e.g. Alyssa" v-model="nameInput" />
        </label>
        <button class="shbtn reg-btn" @click="startNew(null, nameInput, false)">Start new</button>
      </div>
    </BCard>
    <BCard>
      Or, import an existing one
      <div class="formgroup fg-styled">
        <label class="formitem">Upload
          <input type="file" accept="application/json" ref="importInput" />
        </label>
        <button class="shbtn" @click="startImport">
          Import
        </button>
      </div>
    </BCard>
  </div>
  <div class="todo-bc-wrapper novis">
    <BCard>
      <form class="formgroup fg-styled" v-on:submit.prevent="addTodo">
        <label class="formitem">Item
          <input v-model="newTodo" required placeholder="e.g. Whole wheat bread" />
        </label>
        <label class="formitem">Qty
          <input v-model="newQty" required type="number" min="0" value="1" />
        </label>
        <label>Image
          <BFormCheckbox switch v-model="sgImageShow" />
        </label>
        <div class="fg-subgroup" v-show="sgImageShow">
          <label class="formitem">Link
            <input v-model="newImg" placeholder="begins with https:// or data:" :disabled="imageLockState" />
          </label>
          <label class="formitem">Upload
            <input type="file" accept="image/*" @change="onFileChange" ref="fileInput" />
          </label>
          <button type="button" v-if="imageLockState" @click="clearImage" class="shbtn warn-btn">
            Clear Image
          </button>
        </div>
        <button class="shbtn reg-btn w-100">Add</button>
      </form>

      <div class="centered header-fixed info-banner" role="button" tabindex="0" :aria-expanded="showFinalGraph"
        @click="toggleFinalGraph" @keydown.enter.prevent="toggleFinalGraph" @keydown.space.prevent="toggleFinalGraph">
        <h6>Shopping for</h6>
        <h5>
          {{ csname }}
        </h5>
      </div>
      <Transition name="sheet-fade">
        <div v-if="showFinalGraph" class="presentation-backdrop" @click="toggleFinalGraph"></div>
      </Transition>
      <Transition name="sheet-up">
        <div v-if="showFinalGraph" class="presentation-sheet" aria-modal="true">
          <div class="presentation-sheet-inner">
            <ModuleFinalGraph :todos="todos" :customer-name="csname" @toggle-todo="toggleTodoFromPresentation" />
          </div>
        </div>
      </Transition>
      <div class="header-spacer"></div>
      <div class="todo-table">
        <BTable striped bordered hover :items="todos" :fields="fields">
          <template #cell(done)="{ item }">
            <BFormCheckbox v-model="item.done" size="lg" />
          </template>
          <template #cell(text)="{ item }">
            <span :style="{ textDecoration: item.done ? 'line-through' : 'none' }">{{
              item.text
              }}</span>
          </template>
          <template #cell(qty)="{ item }">
            <span :style="{ textDecoration: item.done ? 'line-through' : 'none' }">{{
              item.qty
              }}</span>
          </template>
          <template #cell(img)="{ item }">
            <img v-if="item.img" :class="{ 'bw-filter': item.done }" :src="item.img"
              style="max-width: 100px; max-height: 100px" />
          </template>
          <template #cell(delete)="{ item }">
            <button v-on:click="removeTodo(item)" class="shbtn warn-btn">Delete</button>
          </template>
        </BTable>
      </div>
    </BCard>
    <BCard>
      <div class="list-actions">
        <button class="shbtn" @click="startExport" :disabled="!todos.length">Export</button>
        <button class="shbtn warn-btn" @click="deleteList">Quit List</button>
      </div>
    </BCard>
  </div>
</template>

<style scoped>
.info-banner {
  padding: 1%;
  cursor: pointer;
}

.header-spacer {
  height: 5.5rem;
}

.presentation-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.35);
  backdrop-filter: blur(2px);
  z-index: 105;
}

.presentation-sheet {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 110;
  height: 90vh;
  overflow-y: auto;
  padding: 0.75rem 0.75rem calc(0.75rem + env(safe-area-inset-bottom));
}

.presentation-sheet-inner {
  width: min(960px, calc(100% - 0.2rem));
  margin: 0 auto;
  box-shadow: 0 -14px 34px rgba(0, 0, 0, 0.25);
  border-radius: 20px 20px 0 0;
}

.sheet-fade-enter-active,
.sheet-fade-leave-active {
  transition: opacity 0.2s ease;
}

.sheet-fade-enter-from,
.sheet-fade-leave-to {
  opacity: 0;
}

.sheet-up-enter-active,
.sheet-up-leave-active {
  transition: transform 0.25s ease, opacity 0.25s ease;
}

.sheet-up-enter-from,
.sheet-up-leave-to {
  transform: translateY(100%);
  opacity: 0;
}

.fc-home {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
}

.todo-bc-wrapper {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.todo-table {
  flex: 1;
}

.list-actions {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.formitem {
  display: block;
}

@media (max-width: 768px) {
  .fc-home {
    flex-direction: column;
    align-items: stretch;
  }

  .fc-home :deep(.card) {
    width: 100%;
  }

  .header-spacer {
    height: 5rem;
  }

  .presentation-sheet {
    height: 90vh;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
  }

  .todo-bc-wrapper {
    flex-direction: column;
    align-items: stretch;
  }

  .formgroup {
    flex: auto;
  }

  .todo-table {
    overflow-x: auto;
  }
}
</style>
