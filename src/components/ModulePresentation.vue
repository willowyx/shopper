<script setup lang="js">
import { computed } from 'vue'
import { BFormCheckbox } from 'bootstrap-vue-next'

const props = defineProps({
  todos: {
    type: Array,
    required: true,
  },
  customerName: {
    type: String,
    default: '',
  },
})

const emit = defineEmits(['toggle-todo'])

const totals = computed(() => {
  const totalItems = props.todos.length
  const completedItems = props.todos.filter((todo) => todo.done).length
  const remainingItems = totalItems - completedItems

  const totalQty = props.todos.reduce((sum, todo) => sum + Number(todo.qty || 0), 0)
  const completedQty = props.todos
    .filter((todo) => todo.done)
    .reduce((sum, todo) => sum + Number(todo.qty || 0), 0)

  const itemProgress = totalItems ? Math.round((completedItems / totalItems) * 100) : 0

  return {
    totalItems,
    completedItems,
    remainingItems,
    totalQty,
    completedQty,
    itemProgress,
  }
})

const nextItems = computed(() => props.todos.filter((todo) => !todo.done).slice(0, 2))
const featuredNextItem = computed(() => nextItems.value[0] ?? null)

function onToggleItem(todoId, isDone) {
  emit('toggle-todo', { todoId, isDone })
}
</script>

<template>
  <section class="screen-shell">
    <header class="screen-header">
      <p class="screen-label">Shopping for</p>
      <h5>{{ customerName || 'Shopping List' }}</h5>
    </header>

    <div class="status-row">
      <article class="status-chip done">
        <span class="label">Completed</span>
        <strong>{{ totals.completedItems }} / {{ totals.totalItems }}</strong>
      </article>
      <article class="status-chip remaining">
        <span class="label">Remaining</span>
        <strong>{{ totals.remainingItems }}</strong>
      </article>
    </div>

    <div class="next-row">
      <article class="next-image-card">
        <template v-if="featuredNextItem">
          <img
            v-if="featuredNextItem.img"
            class="next-image"
            :src="featuredNextItem.img"
            :alt="featuredNextItem.text"
          />
          <div v-else class="next-image-empty">No image</div>
          <!-- <p class="next-image-label">Next item</p> -->
        </template>
        <p v-else class="next-image-empty">No queued items</p>
      </article>

      <div class="list-card">
        <h6>Up next</h6>
        <ul v-if="nextItems.length" class="screen-list">
          <li v-for="todo in nextItems" :key="todo.id">
            <span class="item-name">{{ todo.text }}</span>
            <span class="item-meta">x{{ todo.qty }}</span>
          </li>
        </ul>
        <p v-else class="all-done">All items checked</p>
      </div>
    </div>

    <div class="progress-card">
      <div class="progress-head">
        <h6>Shop progress</h6>
        <span>{{ totals.itemProgress }}%</span>
      </div>
      <div class="bar-bg">
        <div class="bar-fill" :style="{ width: `${totals.itemProgress}%` }"></div>
      </div>
    </div>

    <div class="checklist-card">
      <h6>Shopping checklist</h6>
      <ul v-if="todos.length" class="checklist">
        <li v-for="todo in todos" :key="todo.id" :class="{ done: todo.done }">
          <BFormCheckbox :model-value="todo.done" @update:model-value="(val) => onToggleItem(todo.id, val)" />
          <button type="button" class="check-text-btn" @click="onToggleItem(todo.id, !todo.done)">
            {{ todo.text }}
          </button>
          <span class="item-meta">x{{ todo.qty }}</span>
        </li>
      </ul>
      <p v-else class="all-done">No items yet</p>
    </div>
  </section>
</template>

<style scoped>
.screen-shell {
  display: flex;
  flex-direction: column;
  gap: 0.9rem;
  padding: 0.9rem;
  border-radius: 18px;
  border: 1px solid rgba(0, 0, 0, 0.15);
  background: linear-gradient(180deg, #f5fbf5 0%, #ffffff 42%);
  min-height: 100%;
}

.screen-header {
  border-radius: 12px;
  padding: 0.8rem;
  background: #1cbd00;
  color: #fff;
}

.screen-header h5,
.screen-header p {
  margin: 0;
}

.screen-label {
  opacity: 0.85;
  font-size: 0.82rem;
}

.status-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
  gap: 0.6rem;
}

.status-chip {
  border-radius: 12px;
  padding: 0.7rem;
  border: 1px solid rgba(0, 0, 0, 0.12);
  background: #fff;
}

.status-chip .label {
  display: block;
  font-size: 0.78rem;
  color: #5f6873;
}

.status-chip strong {
  font-size: 1rem;
}

.next-row {
  display: grid;
  grid-template-columns: minmax(180px, 220px) minmax(0, 1fr);
  gap: 0.8rem;
}

.progress-card,
.next-image-card,
.list-card,
.checklist-card {
  border-radius: 12px;
  padding: 0.8rem;
  background: #fff;
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.next-image-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 0.6rem;
}

.next-image {
  aspect-ratio: 1 / 1;
  object-fit: cover;
  border-radius: 10px;
  background: #eef3ee;
}

.next-image-label {
  margin: 0;
  font-size: 0.82rem;
  color: #5f6873;
  font-weight: 600;
}

.next-image-empty {
  display: grid;
  place-items: center;
  width: 100%;
  aspect-ratio: 1 / 1;
  margin: 0;
  border-radius: 10px;
  background: #eef3ee;
  color: #5f6873;
  font-weight: 600;
  text-align: center;
}

.progress-card h6,
.next-image-card h6,
.list-card h6,
.checklist-card h6 {
  margin: 0;
}

.progress-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.35rem;
}

.bar-bg {
  width: 100%;
  height: 14px;
  border-radius: 999px;
  overflow: hidden;
  background: #d9e8dd;
}

.bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #1cbd00, #6ad65a);
  transition: width 0.3s ease;
}

.screen-list {
  margin: 0.5rem 0 0;
  padding: 0;
  list-style: none;
}

.screen-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  padding: 0.45rem 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.screen-list li:last-child {
  border-bottom: 0;
  padding-bottom: 0;
}

.item-name {
  font-weight: 600;
}

.item-meta {
  font-size: 0.85rem;
  color: #5f6873;
}

.all-done {
  margin: 0.6rem 0 0;
  font-weight: 600;
}

.checklist {
  margin: 0.5rem 0 0;
  padding: 0;
  list-style: none;
  max-height: 34vh;
  overflow-y: auto;
}

.checklist li {
  display: grid;
  grid-template-columns: auto 1fr auto;
  align-items: center;
  gap: 0.6rem;
  padding: 0.45rem 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.checklist li:last-child {
  border-bottom: 0;
  padding-bottom: 0;
}

.check-text-btn {
  border: 0;
  background: transparent;
  padding: 0;
  margin: 0;
  text-align: left;
  font-weight: 600;
  color: inherit;
  cursor: pointer;
}

.check-text-btn:hover {
  text-decoration: underline;
}

.checklist li.done .check-text-btn,
.checklist li.done .item-meta {
  text-decoration: line-through;
  color: #7a838e;
}

[data-bs-theme='dark'] .screen-shell {
  border-color: rgba(248, 249, 250, 0.22);
  background: linear-gradient(180deg, #131f12 0%, #171a1e 42%);
}

[data-bs-theme='dark'] .status-chip,
[data-bs-theme='dark'] .next-image-card,
[data-bs-theme='dark'] .progress-card,
[data-bs-theme='dark'] .list-card,
[data-bs-theme='dark'] .checklist-card {
  background: #20242a;
  border-color: rgba(248, 249, 250, 0.22);
}

[data-bs-theme='dark'] .status-chip .label,
[data-bs-theme='dark'] .next-image-label,
[data-bs-theme='dark'] .item-meta {
  color: #b0b8c2;
}

[data-bs-theme='dark'] .next-image,
[data-bs-theme='dark'] .next-image-empty {
  background: #2a313b;
}

[data-bs-theme='dark'] .bar-bg {
  background: #303846;
}

[data-bs-theme='dark'] .screen-list li,
[data-bs-theme='dark'] .checklist li {
  border-color: rgba(248, 249, 250, 0.12);
}

[data-bs-theme='dark'] .checklist li.done .check-text-btn,
[data-bs-theme='dark'] .checklist li.done .item-meta {
  color: #95a0ab;
}

@media (max-width: 640px) {
  .next-row {
    grid-template-columns: 1fr;
  }
}
</style>
