<template>
  <HeaderComponent />
  <div class="container">
    <BalanceComponent :total="+total" />
    <IncomeExpensesComponent :income="+income" :expenses="+expenses" />
    <TransactionListComponent :transactions="transactions" @transactionDeleted="handleTransactionDeleted"/>
    <AddTransactionComponent @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>

import HeaderComponent from './components/HeaderComponent.vue';
import BalanceComponent from './components/BalanceComponent.vue';
import IncomeExpensesComponent from './components/IncomeExpensesComponent.vue';
import TransactionListComponent from './components/TransactionListComponent.vue';
import AddTransactionComponent from './components/AddTransactionComponent.vue';

import { ref, computed, onMounted } from "vue";
import { useToast } from "vue-toastification";

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem("transactions"));
  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

const total = computed(() => {
  return transactions.value.reduce((accumulator, transaction) => {
    return accumulator + transaction.amount;
  }, 0);
});

const income = computed(() => {
  return transactions.value
    .filter(transaction => transaction.amount > 0)
    .reduce((accumulator, transaction) => {
      return accumulator + transaction.amount;
    }, 0)
    .toFixed(2);
});

const expenses = computed(() => {
  return transactions.value
    .filter(transaction => transaction.amount < 0)
    .reduce((accumulator, transaction) => {
      return accumulator + transaction.amount;
    }, 0)
    .toFixed(2);
});

// add transaction to the array
const handleTransactionSubmitted = (transactionData) => {
  transactions.value = [...transactions.value, {
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount
  }];
  saveTransactionsToLocalStorage();
  toast.success("Transaction added");
};

const handleTransactionDeleted = (transactionId) => {
  transactions.value = transactions.value.filter(transaction => transaction.id !== transactionId);
  saveTransactionsToLocalStorage();
  toast.success("Transaction deleted");
};

const generateUniqueId = () => {
  return Math.floor(Math.random() * 1000000);
};

const saveTransactionsToLocalStorage = () => {
  localStorage.setItem("transactions", JSON.stringify(transactions.value));
};

</script>