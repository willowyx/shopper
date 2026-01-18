<script setup lang="js">
import { ref, watch } from 'vue'
import { BTable, BFormCheckbox } from 'bootstrap-vue-next'

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
const todos = ref([
  { id: dfid++, text: 'Bread', qty: 1, done: false },
  { id: dfid++, text: 'Eggs', qty: 12, done: false },
  { id: dfid++, text: 'Cheese', qty: 1, done: false },
])

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
    sortable: true,
  },
  {
    key: 'qty',
    label: 'Qty',
    sortable: true,
  },
  {
    key: 'text',
    label: 'Item',
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
  <div class="todo-wrapper">
    <form class="formgroup" v-on:submit.prevent="addTodo">
      <label class="formitem"
        >Item
        <input v-model="newTodo" required placeholder="e.g. Whole wheat bread" />
      </label>
      <label class="formitem"
        >Qty
        <input v-model="newQty" required value="1" />
      </label>
      <label>Image <BFormCheckbox switch v-model="sgImageShow" /></label>
      <div class="fg-subgroup" v-show="sgImageShow">
        <label class="formitem"
          >Link
          <input
            v-model="newImg"
            placeholder="begins with https:// or data:"
            :disabled="imageLockState"
          />
        </label>
        <label class="formitem"
          >Upload
          <input type="file" accept="image/*" @change="onFileChange" ref="fileInput" />
        </label>
        <button type="button" v-if="imageLockState" @click="clearImage" class="shbtn warn-btn">
          Clear Image
        </button>
      </div>
      <button class="shbtn reg-btn w-100">Add</button>
    </form>

    <div class="todo-table">
      <BTable striped bordered hover :items="todos" :fields="fields">
        <template #cell(done)="{ item }">
          <BFormCheckbox v-model="item.done" size="lg" />
        </template>
        <template #cell(qty)="{ item }">
          <span :style="{ textDecoration: item.done ? 'line-through' : 'none' }">{{
            item.qty
          }}</span>
        </template>
        <template #cell(text)="{ item }">
          <span :style="{ textDecoration: item.done ? 'line-through' : 'none' }">{{
            item.text
          }}</span>
        </template>
        <template #cell(img)="{ item }">
          <img
            v-if="item.img"
            :class="{ 'bw-filter': item.done }"
            :src="item.img"
            style="max-width: 100px; max-height: 100px"
          />
        </template>
        <template #cell(delete)="{ item }">
          <button v-on:click="removeTodo(item)" class="shbtn warn-btn">Delete</button>
        </template>
      </BTable>
    </div>
  </div>
</template>

<style scoped>
.todo-wrapper {
  display: flex;
  gap: 20px;
  align-items: flex-start;
}

.todo-table {
  flex: 1;
}

.formitem {
  display: block;
}

@media (max-width: 768px) {
  .todo-wrapper {
    flex-direction: column;
    align-items: stretch;
    margin: 3%;
  }
  .formgroup {
    flex: auto;
    margin: 7%;
  }
  .todo-table {
    overflow-x: auto;
  }
}
</style>
