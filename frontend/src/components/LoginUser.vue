<script setup lang="ts">
import InputGroup from 'primevue/inputgroup'
import InputGroupAddon from 'primevue/inputgroupaddon'
import InputText from 'primevue/inputtext'
import FloatLabel from 'primevue/floatlabel';
import Button from 'primevue/button';
import { useToast } from 'primevue/usetoast'
import { ref } from 'vue';
import axios from 'axios'
import type { User } from '@/types/user'
import store from '@/store'

// TODO: add expiration date of the refresh token
interface LoginResponse {
  user: User
  access_token: string
  refresh_token: string
}

const apiBaseUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000';

const username = ref<string>('')
const password = ref<string>('')
const errorMessage = ref<string>('')
const toast = useToast()

const handleLogin = async () => {
  try {
    const response = await axios.post<LoginResponse>(
      `${apiBaseUrl}/v1/login_user`,
      {
        username: username.value,
        password: password.value
      },
      {
        headers: {
          'Content-Type': 'application/json',
        }
      }
    )
    console.log("Login response data:", response.data);
    store.setUser(response.data.user, response.data.access_token, response.data.refresh_token)
    toast.add({
      severity: 'success',
      summary: `Hello, ${response.data.user.full_name}!`,
      detail: `You have successfully logged in.`,
      life: 3000
    })
    } catch (error: unknown) {
    if (axios.isAxiosError(error)) {
      if (error.response) {
        if (error.response.status === 404) {
          errorMessage.value = error.response.data.message;
        } else if (error.response.status === 401) {
          errorMessage.value = 'Invalid username or password.';
        } else {
          errorMessage.value = 'An error occurred. Please try again later';
        }
      } else {
        errorMessage.value = 'No response received from server.';
      }
    } else {
      errorMessage.value = 'An unexpected error occurred. Please try again.';
    }

    toast.add({
      severity: 'error',
      summary: 'Login failed',
      detail: errorMessage.value,
      life: 3000
    })
  }
}
</script>

<template>  
  <div class="flex flex-column row-gap-5">
    <InputGroup>
      <InputGroupAddon>
        <i class="pi pi-user"></i>
      </InputGroupAddon>
      <FloatLabel>
        <InputText id="username" v-model="username" />
        <label for="username">Username</label>
      </FloatLabel>
    </InputGroup>
    <InputGroup>
      <InputGroupAddon>
        <i class="pi pi-lock"></i>
      </InputGroupAddon>
      <FloatLabel>
        <InputText id="password" type="password" v-model="password" />
        <label for="password">Password</label>
      </FloatLabel>
    </InputGroup>
    <Button label="Login" @click="handleLogin"/>
  </div>
</template>



