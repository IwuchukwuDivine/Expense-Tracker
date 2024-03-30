<template>
  <Header />
  <div class="container">
    <Balance :totalBalance="+totalBalance" />
    <IncomeExpenses :income="+income" :expense="-expense" />
    <History :transactions="transactions" @delete-transaction-id="handleDelete" />
    <AddTransactions @transaction-submitted="handleTransaction" />
  </div>
</template>

<script setup>
import AddTransactions from './components/AddTransactions.vue'
import Balance from './components/Balance.vue'
import Header from './components/Header.vue'
import IncomeExpenses from './components/IncomeExpenses.vue'
import History from './components/History.vue'
import { ref, computed, onMounted } from 'vue'
import { useToast } from 'vue-toastification'

const toast = useToast()

// the data in the transactions should be any data saved in the local storage or an empty array if local storage is empty
const transactions = ref(JSON.parse(localStorage.getItem('transactions')) || [])

// function to save transactions in local storage whenever there is a change
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value))
}

// get total balance
const totalBalance = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount
  }, 0)
})

// get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount
    }, 0)
    .toFixed(2)
})
// get expense
const expense = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount
    }, 0)
    .toFixed(2)
})

// add transaction
function handleTransaction(transactionData) {
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount
  })

  saveTransactionsToLocalStorage()

  toast.success('Transaction added')
}

function generateUniqueId() {
  return Math.floor(Math.random() * 2000000)
}

// to delete transaction
function handleDelete(transactionId) {
  transactions.value = transactions.value.filter((transaction) => transaction.id != transactionId)

  saveTransactionsToLocalStorage()

  toast.success('Transaction deleted')
}

//watch for changes in local storage and save to local storage
onMounted(() => {
  transactions.value = JSON.parse(localStorage.getItem('transactions')) || []
})
</script>

<style scoped></style>
