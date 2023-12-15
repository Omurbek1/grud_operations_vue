<template>
  <div v-for="user in users" :key="user.id">
    <p>{{ user.name }}</p>
    <p>{{ user.email }}</p>
    <p>{{ user.phone }}</p>
    <p>{{ user.website }}</p>
    <hr />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";

interface User {
  id: number;
  name: string;
  email: string;
  phone: string;
  website: string;
}
const users = ref<User[]>([]);

const getUsers = async () => {
  return fetch("https://jsonplaceholder.typicode.com/users").then((response) =>
    response.json()
  );
};
onMounted(async () => {
  getUsers().then((data) => (users.value = data));
});
</script>

<style scoped></style>
