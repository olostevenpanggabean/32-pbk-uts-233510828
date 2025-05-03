<script setup>
import { ref, onMounted } from 'vue';

const days = ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat', 'Sabtu', 'Minggu'];
const selectedDay = ref('Senin');
const filter = ref('all');
const expenses = ref({});
const newExpense = ref({
  text: '',
  amount: 0,
  completed: false
});

days.forEach(day => {
  expenses.value[day] = [];
});

function addExpense() {
  if (!newExpense.value.text || newExpense.value.amount <= 0) return;
  
  expenses.value[selectedDay.value].push({
    ...newExpense.value,
    id: Date.now()
  });
  
  newExpense.value = { text: '', amount: 0, completed: false };
  saveToLocalStorage();
}

function removeExpense(day, index) {
  expenses.value[day].splice(index, 1);
  saveToLocalStorage();
}

function filteredExpenses(day) {
  const dayExpenses = expenses.value[day];
  return filter.value === 'uncompleted' 
    ? dayExpenses.filter(exp => !exp.completed)
    : dayExpenses;
}

function calculateDayTotal(day) {
  return expenses.value[day].reduce((total, exp) => total + exp.amount, 0);
}

function calculateWeekTotal() {
  return days.reduce((total, day) => total + calculateDayTotal(day), 0);
}

function saveToLocalStorage() {
  localStorage.setItem('kosExpenses', JSON.stringify(expenses.value));
}

function loadFromLocalStorage() {
  const saved = localStorage.getItem('kosExpenses');
  if (saved) {
    expenses.value = JSON.parse(saved);
  }
}

onMounted(() => {
  loadFromLocalStorage();
});
</script>

<template>
  <div class="expense-tracker">
    <h1>Pengeluaran Anak Kos</h1>
    
    <div class="controls">
      <select v-model="selectedDay">
        <option v-for="day in days" :value="day">{{ day }}</option>
      </select>
      <input 
        v-model="newExpense.text" 
        placeholder="Misal: Makan siang..."
        @keyup.enter="addExpense"
      >
      <input
        v-model.number="newExpense.amount"
        type="number"
        placeholder="Rp"
      >
      <button @click="addExpense">Tambah</button>
    </div>

    <div class="filter-buttons">
      <button @click="filter = 'all'" :class="{ active: filter === 'all' }">Semua</button>
      <button @click="filter = 'uncompleted'" :class="{ active: filter === 'uncompleted' }">Belum Dibayar</button>
    </div>

    <div v-for="day in days" :key="day" class="day-section">
      <h2>{{ day }}</h2>
      <ul>
        <li 
          v-for="(expense, index) in filteredExpenses(day)" 
          :key="index"
          :class="{ completed: expense.completed }"
        >
          <input 
            type="checkbox" 
            v-model="expense.completed"
            @change="saveToLocalStorage"
          >
          <span class="expense-text">
            {{ expense.text }} 
            <span class="amount">Rp {{ expense.amount.toLocaleString() }}</span>
          </span>
          <button @click="removeExpense(day, index)" class="delete-btn">✕</button>
        </li>
      </ul>
      <div class="day-total">
        Total: Rp {{ calculateDayTotal(day).toLocaleString() }}
      </div>
    </div>

    <div class="week-total">
      <h3>Total Pengeluaran Minggu Ini:</h3>
      <p>Rp {{ calculateWeekTotal().toLocaleString() }}</p>
    </div>
  </div>
</template>

<style>
.expense-tracker {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.controls select, .controls input {
  padding: 8px 12px;
  border: 1px solid #e5e4e4;
  border-radius: 4px;
}

.controls input[type="number"] {
  width: 80px;
}

button {
  padding: 8px 16px;
  background-color: #A5A58D;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.filter-buttons {
  margin-bottom: 20px;
}

.filter-buttons button {
  background-color:     whitesmoke;
  color: #671c1c;
  margin-right: 10px;
}

.filter-buttons button.active {
  background-color: #A5A58D;
  color: white;
}

.day-section {
  background-color: blanchedalmond;
  color: #671c1c;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 15px;
}

.day-section h2 {
  margin-top: 0;
  color: #671c1c;
  border-bottom: 1px solid #671c1c;
  padding-bottom: 5px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  display: flex;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid #2F4F4F;
}

.completed .expense-text {
  text-decoration: line-through;
  color: #2F4F4F;
}

.amount {
  font-weight: bold;
  color: black;
  margin-left: 10px;
}

.delete-btn {
  margin-left: auto;
  background-color: #671c1c;
  padding: 2px 8px;
  border-radius: 50%;
}

.day-total {
  text-align: right;
  font-weight: bold;
  margin-top: 10px;
}

.week-total {
  background-color: #A5A58D;
  color:whitesmoke;
  padding: 15px;
  border-radius: 8px;
  text-align: center;
}

.week-total h3 {
  margin-top: 0;
}

.week-total p {
  font-size: 24px;
  margin-bottom: 0;
}
</style>