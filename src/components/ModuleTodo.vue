<script setup lang="js">
import { ref, watch } from 'vue'
import { BTable, BFormCheckbox, BCard } from 'bootstrap-vue-next'
import { version } from '../../package.json'

let dfid = 0

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
const nameInput = ref('')
const todos = ref([
  { id: dfid++, text: 'Paper plates', qty: 1, img: 'noitem.png', done: true },
  { id: dfid++, text: 'Eggs', qty: 12, img: 'noitem.png', done: false },
])
const csname = ref('Alyssa')
const isInit = ref(false)

watch(isInit, (newValue) => {
  if (newValue) {
    for (let i = 0; i < document.getElementsByClassName('todo-bc-wrapper').length; i++) {
      document.getElementsByClassName('todo-bc-wrapper')[i].classList.remove('novis')
    }
    for (let i = 0; i < document.getElementsByClassName('fc-home').length; i++) {
      document.getElementsByClassName('fc-home')[i].classList.add('novis')
    }
  }
})

function startNew(list, name, isImport) {
  if (list) {
    todos.value = list
  }
  if (name) {
    csname.value = name
  } else if (isImport) {
    csname.value = '' // fill name from import
  }
  isInit.value = true
}

function startExport() {
  const exportData = {
    dataName: 'shopper-data',
    gameVersion: version,
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
    <BCard class="novis">
      Or, import an existing one
      <div class="formgroup fg-styled">
        <label class="formitem">Upload
          <input type="file" accept="application/json" ref="importInput" />
        </label>
        <button class="shbtn" @click="
          startNew(
            (list = importInput),
            (name = JSON.parse(importInput.value.files[0].text(csname))),
            (isImport = true),
          )
          ">
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
          <input v-model="newQty" required value="1" />
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

      <div class="centered header-fixed info-banner">
        <h6>Shopping for</h6>
        <h5>
          {{ csname }}
        </h5>
      </div>
      <br /><br />
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
      Export list:
      <button class="shbtn" @click="startExport()" :disabled="!todos.length">Export</button>
    </BCard>
  </div>
</template>

<style scoped>
.info-banner {
  padding: 1%;
  cursor: pointer;
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

.formitem {
  display: block;
}

@media (max-width: 768px) {
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
