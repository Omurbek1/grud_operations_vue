<template>
  <a-button
    type="primary"
    @click="toggleForm"
    style="position: absolute; right: 20px"
    >Add Users</a-button
  >
  <a-modal v-model:open="showForm.showForm" title="Add Users">
    <a-form @submit.prevent="addUsers">
      <a-form-item label="Name">
        <a-input v-model="newUser.name" placeholder="Name" />
      </a-form-item>
      <a-form-item label="Email">
        <a-input v-model="newUser.email" placeholder="Email" />
      </a-form-item>
      <a-form-item label="Phone">
        <a-input v-model="newUser.phone" placeholder="Phone" />
      </a-form-item>
      <a-form-item label="Website">
        <a-input v-model="newUser.website" placeholder="Website" />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" html-type="submit"> Submit </a-button>
      </a-form-item>
    </a-form>
  </a-modal>
  <a-space :size="[40, 20]" wrap style="margin-top: 10px">
    <a-card
      v-for="newUser in users"
      title="Default size card"
      style="width: 300px; margin: 10px"
    >
      <template #extra>
        <a-row>
          <div style="cursor: pointer" @click="openEditDialog(newUser)">
            <EditOutlined />
          </div>
          <div style="cursor: pointer">
            <DeleteOutlined @click="openConfirmDialog(newUser.id)" />
          </div>
        </a-row>
      </template>
      <p>{{ newUser.name }}</p>
      <p>{{ newUser.email }}</p>
      <p>{{ newUser.phone }}</p>
      <p>{{ newUser.website }}</p>
    </a-card>
  </a-space>
  <a-modal
    v-model:open="isConfirmDialogVisible"
    title="Delete User"
    ok-text="Delete"
    @ok="deleteUser(newUser.id)"
  >
    <p>Are you sure you want to delete this user?</p>
  </a-modal>
  <a-modal
    v-model:visible="isEditDialogVisible"
    ok-text="Save"
    cancel-text="Cancel"
    @ok="EditUser(newUser)"
  >
    <template #title>
      <a-icon type="edit" />
      Edit User
    </template>
    <a-form-model :model="newUser">
      <a-form-model-item label="Name" prop="name">
        <a-input v-model="newUser.name" />
      </a-form-model-item>
      <a-form-model-item label="Email" prop="email">
        <a-input v-model="newUser.email" />
      </a-form-model-item>
      <a-form-model-item label="Phone" prop="phone">
        <a-input v-model="newUser.phone" />
      </a-form-model-item>
      <a-form-model-item label="Website" prop="website">
        <a-input v-model="newUser.website" />
      </a-form-model-item>
    </a-form-model>
  </a-modal>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { DeleteOutlined, EditOutlined } from "@ant-design/icons-vue";
import { notification } from "ant-design-vue";
interface User {
  id: number;
  name: string;
  email: string;
  phone: string;
  website: string;
}
interface showFormInterface {
  showForm: boolean;
}
const users = ref<User[]>([]);
const showForm = ref<showFormInterface>({ showForm: false });
const newUser = ref<User>({
  id: 0,
  name: "",
  email: "",
  phone: "",
  website: "",
});
const isConfirmDialogVisible = ref(false);
const isEditDialogVisible = ref(false);

const openEditDialog = (user: any) => {
  console.log(user);
  newUser.value = user;
  console.log(newUser);
  isEditDialogVisible.value = true;
};
const openConfirmDialog = (id: number) => {
  newUser.value = users.value.find((user) => user.id === id)!;
  isConfirmDialogVisible.value = true;
  console.log(newUser.value);
};
const getUsers = async () => {
  return fetch("https://jsonplaceholder.typicode.com/users").then((response) =>
    response.json()
  );
};
const toggleForm = () => {
  showForm.value.showForm = !showForm.value.showForm;
};
const addUsers = async () => {
  const response = await fetch("https://jsonplaceholder.typicode.com/users", {
    method: "POST",
    body: JSON.stringify(newUser.value),
    headers: {
      "Content-type": "application/json; charset=UTF-8",
    },
  });
  if (response.ok) {
    notification.success({ message: "User added successfully" });
    showForm.value.showForm = false;
    getUsers().then((data) => (users.value = data));
  }
  if (!response.ok) {
    throw new Error("Something went wrong");
  }
  const user = await response.json();

  users.value.push(user);

  newUser.value = {
    id: 0,
    name: "",
    email: "",
    phone: "",
    website: "",
  };
  showForm.value.showForm = false;
};

const EditUser = async (newUser: any) => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/users/${newUser.value.id}`,
    {
      method: "PUT",
      body: JSON.stringify(newUser.value),
      headers: {
        "Content-type": "application/json; charset=UTF-8",
      },
    }
  );
  if (response.ok) {
    notification.success({ message: "User updated successfully" });
    showForm.value.showForm = false;
    getUsers().then((data) => (users.value = data));
  }
  if (!response.ok) {
    throw new Error("Something went wrong");
    notification.error({ message: "Something went wrong" });
  }
};
const deleteUser = async (id: number) => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/users/${id}`,
    {
      method: "DELETE",
    }
  );
  if (response.ok) {
    users.value = users.value.filter((user) => user.id !== id);
    isConfirmDialogVisible.value = false;
    notification.success({ message: "User deleted successfully" });
  }
  if (!response.ok) {
    throw new Error("Something went wrong");
  }
};
onMounted(async () => {
  getUsers().then((data) => (users.value = data));
  return {
    users,
    getUsers,
    toggleForm,
    addUsers,
  };
});
components: {
  //  'a-icon': DeleteOutlined
}
</script>

<style scoped></style>
