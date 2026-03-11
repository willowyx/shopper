<script setup lang="js">
import { computed } from 'vue'

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

const nextItems = computed(() => props.todos.filter((todo) => !todo.done).slice(0, 4))
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

    <div class="progress-card">
      <div class="progress-head">
        <h6>Shop progress</h6>
        <span>{{ totals.itemProgress }}%</span>
      </div>
      <div class="bar-bg">
        <div class="bar-fill" :style="{ width: `${totals.itemProgress}%` }"></div>
      </div>
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

.progress-card,
.list-card {
  border-radius: 12px;
  padding: 0.8rem;
  background: #fff;
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.progress-card h6,
.list-card h6 {
  margin: 0;
}

.progress-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.35rem;
}

.progress-head.second {
  margin-top: 0.75rem;
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

.bar-fill.qty {
  background: linear-gradient(90deg, #0d6efd, #5aa3ff);
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

.screen-list.muted .item-name,
.screen-list.muted .item-meta {
  color: #7a838e;
  text-decoration: line-through;
}

.all-done {
  margin: 0.6rem 0 0;
  font-weight: 600;
}

[data-bs-theme='dark'] .screen-shell {
  border-color: rgba(248, 249, 250, 0.22);
  background: linear-gradient(180deg, #131f12 0%, #171a1e 42%);
}

[data-bs-theme='dark'] .status-chip,
[data-bs-theme='dark'] .progress-card,
[data-bs-theme='dark'] .list-card {
  background: #20242a;
  border-color: rgba(248, 249, 250, 0.22);
}

[data-bs-theme='dark'] .status-chip .label,
[data-bs-theme='dark'] .item-meta {
  color: #b0b8c2;
}

[data-bs-theme='dark'] .bar-bg {
  background: #303846;
}

[data-bs-theme='dark'] .screen-list li {
  border-color: rgba(248, 249, 250, 0.12);
}

[data-bs-theme='dark'] .screen-list.muted .item-name,
[data-bs-theme='dark'] .screen-list.muted .item-meta {
  color: #95a0ab;
}
</style>
