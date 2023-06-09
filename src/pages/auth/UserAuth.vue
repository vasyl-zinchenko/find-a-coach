<template>
  <div>
    <base-dialog :show="!!error" title="An error occurred" @close="handleError">
      <p>{{ error }}</p>
    </base-dialog>
    <base-dialog :show="isLoading" title="Authenticating..." fixed>
      <base-spinner></base-spinner>
    </base-dialog>

    <base-card>
      <form @submit.prevent="submitForm">
        <div class="form-control">
          <label for="email">E-Mail</label>
          <input type="email" id="email" v-model.trim="authStore.email" />
        </div>

        <div class="form-control">
          <label for="password">Password</label>
          <input
            type="password"
            id="password"
            v-model.trim="authStore.password"
          />
        </div>

        <p v-if="!formIsValid">
          Please enter a valid email and password (must be at least 6 characters
          long).
        </p>

        <base-button>{{ submitButtonCaption }}</base-button>

        <base-button type="button" mode="flat" @click="switchAuthMode">
          {{ switchModeButtonCaption }}
        </base-button>
      </form>
    </base-card>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useAuthStore } from '../../stores/auth';
import { useRouter, useRoute } from 'vue-router';
const router = useRouter();
const route = useRoute();

const authStore = useAuthStore();

let formIsValid = ref(true);
const mode = ref('login');
const isLoading = ref(false);
const error = ref(null);

const submitButtonCaption = computed(() => {
  if (mode.value === 'login') {
    return 'Login';
  } else {
    return 'Signup';
  }
});

const switchModeButtonCaption = computed(() => {
  if (mode.value === 'login') {
    return 'Signup instead';
  } else {
    return 'Login instead';
  }
});

async function submitForm() {
  formIsValid.value = true;

  if (
    authStore.email === '' ||
    !authStore.email.includes('@') ||
    authStore.password.length < 6
  ) {
    formIsValid.value = false;
    return;
  }

  isLoading.value = true;

  try {
    if (mode.value === 'login') {
      await authStore.login();
    } else {
      await authStore.signup();
    }
    const redirectUrl = route.query.redirect || '/coaches';
    router.replace(redirectUrl);
  } catch (err) {
    error.value = err.message || 'Failed to authenticate, try later';
  }

  isLoading.value = false;
}

function switchAuthMode() {
  if (mode.value === 'login') {
    mode.value = 'signup';
  } else {
    mode.value = 'login';
  }
}

function handleError() {
  error.value = null;
}
</script>

<style scoped>
form {
  margin: 1rem;
  padding: 1rem;
}

.form-control {
  margin: 0.5rem 0;
}

label {
  font-weight: bold;
  margin-bottom: 0.5rem;
  display: block;
}

input,
textarea {
  display: block;
  width: 100%;
  font: inherit;
  border: 1px solid #ccc;
  padding: 0.15rem;
}

input:focus,
textarea:focus {
  border-color: #3d008d;
  background-color: #faf6ff;
  outline: none;
}
</style>
