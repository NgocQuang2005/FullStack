# Hướng Dẫn Toàn Diện Vue.js 3 & Ecosystem

## 🎯 Roadmap Học Vue.js 3

### Phase 1: Nền Tảng (2-3 tuần)
1. **JavaScript ES6+** (Prerequisite)
2. **Vue.js 3 Core Concepts**
3. **Composition API**
4. **Reactivity System**

### Phase 2: Ecosystem Core (3-4 tuần)
1. **Vue Router 4**
2. **Pinia State Management**
3. **Component Communication**
4. **Forms & Validation**

### Phase 3: Advanced & Tools (2-3 tuần)
1. **PrimeVue UI Framework**
2. **Axios HTTP Client**
3. **AJV Validation**
4. **Testing & Deployment**

---

## 📚 PHẦN 1: VUE.JS 3 CORE

### 1.1 Setup & Installation

```bash
# Tạo project mới
npm create vue@latest my-vue-project
cd my-vue-project
npm install

# Hoặc sử dụng Vite
npm create vite@latest my-vue-app -- --template vue
```

### 1.2 Composition API - Cốt Lõi

#### Basic Setup Function
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Count: {{ count }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>

<script>
import { ref, reactive, computed, watch, onMounted } from 'vue'

export default {
  name: 'CounterComponent',
  setup() {
    // Reactive references
    const count = ref(0)
    const title = ref('Vue 3 Counter')
    
    // Reactive object
    const state = reactive({
      user: {
        name: 'John',
        email: 'john@example.com'
      },
      isLoading: false
    })
    
    // Computed properties
    const doubleCount = computed(() => count.value * 2)
    const isEven = computed(() => count.value % 2 === 0)
    
    // Methods
    const increment = () => {
      count.value++
    }
    
    const decrement = () => {
      count.value--
    }
    
    // Watchers
    watch(count, (newVal, oldVal) => {
      console.log(`Count changed from ${oldVal} to ${newVal}`)
    })
    
    // Lifecycle hooks
    onMounted(() => {
      console.log('Component mounted!')
    })
    
    // Return để expose ra template
    return {
      count,
      title,
      state,
      doubleCount,
      isEven,
      increment,
      decrement
    }
  }
}
</script>
```

#### Script Setup Syntax (Recommended)
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <UserCard :user="user" @update-user="handleUserUpdate" />
    <ProductList :products="products" />
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue'
import UserCard from './UserCard.vue'
import ProductList from './ProductList.vue'

// Reactive data
const title = ref('My App')
const user = reactive({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com'
})

const products = ref([])

// Computed
const userDisplayName = computed(() => {
  return `${user.name} (${user.email})`
})

// Methods
const handleUserUpdate = (updatedUser) => {
  Object.assign(user, updatedUser)
}

const fetchProducts = async () => {
  try {
    const response = await fetch('/api/products')
    products.value = await response.json()
  } catch (error) {
    console.error('Failed to fetch products:', error)
  }
}

// Watchers
watch(() => user.name, (newName) => {
  console.log('User name changed:', newName)
})

// Lifecycle
onMounted(() => {
  fetchProducts()
})
</script>
```

### 1.3 Reactivity System Deep Dive

#### ref vs reactive
```javascript
import { ref, reactive, toRefs, isRef, unref } from 'vue'

// ref: cho primitive values và objects
const count = ref(0)
const user = ref({ name: 'John', age: 25 })

// reactive: chỉ cho objects
const state = reactive({
  count: 0,
  user: { name: 'John', age: 25 }
})

// toRefs: convert reactive object thành refs
const { count: reactiveCount, user: reactiveUser } = toRefs(state)

// Utility functions
console.log(isRef(count)) // true
console.log(unref(count)) // 0 (same as count.value)
```

#### Computed Properties Advanced
```javascript
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// Getter only
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})

// Getter + Setter
const fullNameWritable = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(value) {
    [firstName.value, lastName.value] = value.split(' ')
  }
})
```

### 1.4 Component Communication

#### Props & Emits
```vue
<!-- Child Component: UserCard.vue -->
<template>
  <div class="user-card">
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    <button @click="editUser">Edit</button>
  </div>
</template>

<script setup>
// Props definition
const props = defineProps({
  user: {
    type: Object,
    required: true,
    validator: (user) => {
      return user && typeof user.name === 'string'
    }
  },
  editable: {
    type: Boolean,
    default: true
  }
})

// Emits definition
const emit = defineEmits(['update-user', 'delete-user'])

const editUser = () => {
  if (props.editable) {
    emit('update-user', { ...props.user, name: 'Updated Name' })
  }
}
</script>
```

#### Provide/Inject
```vue
<!-- Parent Component -->
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
const user = ref({ name: 'John', role: 'admin' })

provide('theme', theme)
provide('currentUser', user)
</script>

<!-- Child Component (any level deep) -->
<script setup>
import { inject } from 'vue'

const theme = inject('theme', 'light') // default value
const currentUser = inject('currentUser')
</script>
```

---

## 📚 PHẦN 2: VUE ROUTER 4

### 2.1 Basic Setup
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: About
  },
  {
    path: '/users/:id',
    name: 'UserDetail',
    component: () => import('../views/UserDetail.vue'),
    props: true
  },
  {
    path: '/admin',
    component: () => import('../layouts/AdminLayout.vue'),
    children: [
      {
        path: '',
        name: 'AdminDashboard',
        component: () => import('../views/admin/Dashboard.vue')
      },
      {
        path: 'users',
        name: 'AdminUsers',
        component: () => import('../views/admin/Users.vue')
      }
    ]
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### 2.2 Navigation & Route Guards
```vue
<script setup>
import { useRouter, useRoute, onBeforeRouteLeave } from 'vue-router'

const router = useRouter()
const route = useRoute()

// Programmatic navigation
const goToUser = (userId) => {
  router.push({ name: 'UserDetail', params: { id: userId } })
}

const goBack = () => {
  router.back()
}

// Route guards
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    return confirm('You have unsaved changes. Leave anyway?')
  }
})

// Access route params/query
console.log(route.params.id)
console.log(route.query.tab)
</script>
```

---

## 📚 PHẦN 3: PINIA STATE MANAGEMENT

### 3.1 Store Definition
```javascript
// stores/user.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref(null)
  const users = ref([])
  const loading = ref(false)
  const error = ref(null)
  
  // Getters (computed)
  const isLoggedIn = computed(() => !!user.value)
  const userName = computed(() => user.value?.name || 'Guest')
  const adminUsers = computed(() => 
    users.value.filter(u => u.role === 'admin')
  )
  
  // Actions
  async function login(credentials) {
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      })
      
      if (!response.ok) {
        throw new Error('Login failed')
      }
      
      user.value = await response.json()
      localStorage.setItem('token', user.value.token)
    } catch (err) {
      error.value = err.message
      throw err
    } finally {
      loading.value = false
    }
  }
  
  function logout() {
    user.value = null
    localStorage.removeItem('token')
  }
  
  async function fetchUsers() {
    loading.value = true
    try {
      const response = await fetch('/api/users')
      users.value = await response.json()
    } catch (err) {
      error.value = err.message
    } finally {
      loading.value = false
    }
  }
  
  return {
    // State
    user,
    users,
    loading,
    error,
    // Getters
    isLoggedIn,
    userName,
    adminUsers,
    // Actions
    login,
    logout,
    fetchUsers
  }
})
```

### 3.2 Using Store in Components
```vue
<template>
  <div>
    <div v-if="userStore.loading">Loading...</div>
    <div v-else-if="userStore.isLoggedIn">
      <h1>Welcome, {{ userStore.userName }}!</h1>
      <button @click="userStore.logout">Logout</button>
    </div>
    <div v-else>
      <LoginForm @submit="handleLogin" />
    </div>
  </div>
</template>

<script setup>
import { useUserStore } from '@/stores/user'
import LoginForm from '@/components/LoginForm.vue'

const userStore = useUserStore()

const handleLogin = async (credentials) => {
  try {
    await userStore.login(credentials)
  } catch (error) {
    console.error('Login error:', error)
  }
}
</script>
```

---

## 📚 PHẦN 4: AXIOS HTTP CLIENT

### 4.1 Basic Setup & Interceptors
```javascript
// api/index.js
import axios from 'axios'

const api = axios.create({
  baseURL: process.env.VUE_APP_API_URL || 'http://localhost:3000/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// Request interceptor
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

// Response interceptor
api.interceptors.response.use(
  (response) => {
    return response
  },
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default api
```

### 4.2 API Service Layer
```javascript
// services/userService.js
import api from '@/api'

export const userService = {
  async getUsers(params = {}) {
    const response = await api.get('/users', { params })
    return response.data
  },
  
  async getUser(id) {
    const response = await api.get(`/users/${id}`)
    return response.data
  },
  
  async createUser(userData) {
    const response = await api.post('/users', userData)
    return response.data
  },
  
  async updateUser(id, userData) {
    const response = await api.put(`/users/${id}`, userData)
    return response.data
  },
  
  async deleteUser(id) {
    await api.delete(`/users/${id}`)
  },
  
  async uploadAvatar(id, file) {
    const formData = new FormData()
    formData.append('avatar', file)
    
    const response = await api.post(`/users/${id}/avatar`, formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
    return response.data
  }
}
```

### 4.3 Composable cho API Calls
```javascript
// composables/useApi.js
import { ref } from 'vue'

export function useApi(apiFunction) {
  const data = ref(null)
  const loading = ref(false)
  const error = ref(null)
  
  const execute = async (...args) => {
    loading.value = true
    error.value = null
    
    try {
      const result = await apiFunction(...args)
      data.value = result
      return result
    } catch (err) {
      error.value = err
      throw err
    } finally {
      loading.value = false
    }
  }
  
  return {
    data,
    loading,
    error,
    execute
  }
}

// Usage in component
import { userService } from '@/services/userService'
import { useApi } from '@/composables/useApi'

const { data: users, loading, error, execute: fetchUsers } = useApi(userService.getUsers)

// Fetch data
await fetchUsers({ page: 1, limit: 10 })
```

---

## 📚 PHẦN 5: AJV VALIDATION

### 5.1 Basic Setup & Schema
```javascript
// utils/validation.js
import Ajv from 'ajv'
import addFormats from 'ajv-formats'

const ajv = new Ajv({ allErrors: true, removeAdditional: true })
addFormats(ajv)

// User schema
const userSchema = {
  type: 'object',
  properties: {
    name: {
      type: 'string',
      minLength: 2,
      maxLength: 50,
      pattern: '^[A-Za-z\\s]+$'
    },
    email: {
      type: 'string',
      format: 'email'
    },
    age: {
      type: 'integer',
      minimum: 18,
      maximum: 120
    },
    password: {
      type: 'string',
      minLength: 8,
      pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]'
    },
    role: {
      type: 'string',
      enum: ['user', 'admin', 'moderator']
    },
    address: {
      type: 'object',
      properties: {
        street: { type: 'string' },
        city: { type: 'string' },
        zipCode: { type: 'string', pattern: '^\\d{5}$' }
      },
      required: ['street', 'city'],
      additionalProperties: false
    }
  },
  required: ['name', 'email', 'password'],
  additionalProperties: false
}

export const validateUser = ajv.compile(userSchema)

export function getValidationErrors(validate, data) {
  const isValid = validate(data)
  if (!isValid) {
    return validate.errors.map(error => ({
      field: error.instancePath.slice(1) || error.params?.missingProperty || 'root',
      message: error.message,
      value: error.data
    }))
  }
  return []
}
```

### 5.2 Form Validation Composable
```javascript
// composables/useFormValidation.js
import { ref, computed } from 'vue'
import { validateUser, getValidationErrors } from '@/utils/validation'

export function useFormValidation(validator, initialData = {}) {
  const formData = ref({ ...initialData })
  const errors = ref([])
  const touched = ref(new Set())
  
  const isValid = computed(() => errors.value.length === 0)
  const hasErrors = computed(() => errors.value.length > 0)
  
  const validate = () => {
    errors.value = getValidationErrors(validator, formData.value)
    return isValid.value
  }
  
  const validateField = (fieldName) => {
    touched.value.add(fieldName)
    const fieldErrors = getValidationErrors(validator, formData.value)
      .filter(error => error.field === fieldName)
    
    // Remove old errors for this field
    errors.value = errors.value.filter(error => error.field !== fieldName)
    // Add new errors
    errors.value.push(...fieldErrors)
  }
  
  const getFieldError = (fieldName) => {
    return errors.value.find(error => error.field === fieldName)?.message
  }
  
  const isFieldTouched = (fieldName) => {
    return touched.value.has(fieldName)
  }
  
  const resetForm = () => {
    formData.value = { ...initialData }
    errors.value = []
    touched.value.clear()
  }
  
  return {
    formData,
    errors,
    isValid,
    hasErrors,
    validate,
    validateField,
    getFieldError,
    isFieldTouched,
    resetForm
  }
}
```

### 5.3 Form Component với Validation
```vue
<template>
  <form @submit.prevent="handleSubmit" class="user-form">
    <div class="form-group">
      <label for="name">Name:</label>
      <input
        id="name"
        v-model="formData.name"
        type="text"
        :class="{ error: getFieldError('name') && isFieldTouched('name') }"
        @blur="validateField('name')"
      />
      <span v-if="getFieldError('name') && isFieldTouched('name')" class="error-message">
        {{ getFieldError('name') }}
      </span>
    </div>
    
    <div class="form-group">
      <label for="email">Email:</label>
      <input
        id="email"
        v-model="formData.email"
        type="email"
        :class="{ error: getFieldError('email') && isFieldTouched('email') }"
        @blur="validateField('email')"
      />
      <span v-if="getFieldError('email') && isFieldTouched('email')" class="error-message">
        {{ getFieldError('email') }}
      </span>
    </div>
    
    <div class="form-group">
      <label for="password">Password:</label>
      <input
        id="password"
        v-model="formData.password"
        type="password"
        :class="{ error: getFieldError('password') && isFieldTouched('password') }"
        @blur="validateField('password')"
      />
      <span v-if="getFieldError('password') && isFieldTouched('password')" class="error-message">
        {{ getFieldError('password') }}
      </span>
    </div>
    
    <button type="submit" :disabled="!isValid">
      Create User
    </button>
  </form>
</template>

<script setup>
import { useFormValidation } from '@/composables/useFormValidation'
import { validateUser } from '@/utils/validation'

const emit = defineEmits(['submit'])

const {
  formData,
  isValid,
  validate,
  validateField,
  getFieldError,
  isFieldTouched,
  resetForm
} = useFormValidation(validateUser, {
  name: '',
  email: '',
  password: '',
  role: 'user'
})

const handleSubmit = () => {
  if (validate()) {
    emit('submit', { ...formData.value })
    resetForm()
  }
}
</script>

<style scoped>
.form-group {
  margin-bottom: 1rem;
}

.error {
  border-color: #ff0000;
}

.error-message {
  color: #ff0000;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}
</style>
```

---

## 📚 PHẦN 6: PRIMEVUE UI FRAMEWORK

### 6.1 Setup & Configuration
```javascript
// main.js
import { createApp } from 'vue'
import PrimeVue from 'primevue/config'
import App from './App.vue'

// Components
import Button from 'primevue/button'
import InputText from 'primevue/inputtext'
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import Dialog from 'primevue/dialog'
import Toast from 'primevue/toast'
import ToastService from 'primevue/toastservice'

// Styles
import 'primevue/resources/themes/lara-light-blue/theme.css'
import 'primevue/resources/primevue.min.css'
import 'primeicons/primeicons.css'

const app = createApp(App)

app.use(PrimeVue)
app.use(ToastService)

// Register components globally
app.component('Button', Button)
app.component('InputText', InputText)
app.component('DataTable', DataTable)
app.component('Column', Column)
app.component('Dialog', Dialog)
app.component('Toast', Toast)

app.mount('#app')
```

### 6.2 Advanced DataTable Example
```vue
<template>
  <div class="user-management">
    <div class="toolbar">
      <Button 
        label="New User" 
        icon="pi pi-plus" 
        class="p-button-success"
        @click="openNew"
      />
      <Button 
        label="Delete Selected" 
        icon="pi pi-trash" 
        class="p-button-danger"
        :disabled="!selectedUsers || !selectedUsers.length"
        @click="confirmDeleteSelected"
      />
    </div>
    
    <DataTable
      v-model:selection="selectedUsers"
      :value="users"
      :paginator="true"
      :rows="10"
      :rowsPerPageOptions="[5, 10, 20, 50]"
      :loading="loading"
      dataKey="id"
      :filters="filters"
      filterDisplay="menu"
      :globalFilterFields="['name', 'email', 'role']"
      responsiveLayout="scroll"
      selectionMode="multiple"
      @row-select="onRowSelect"
      @row-unselect="onRowUnselect"
    >
      <template #header>
        <div class="flex justify-content-between">
          <h3>Users</h3>
          <IconField iconPosition="left">
            <InputIcon class="pi pi-search" />
            <InputText 
              v-model="filters['global'].value" 
              placeholder="Global Search"
            />
          </IconField>
        </div>
      </template>
      
      <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
      
      <Column field="name" header="Name" sortable>
        <template #body="{ data }">
          <div class="flex align-items-center">
            <Avatar 
              :image="data.avatar" 
              :label="data.name.charAt(0)"
              class="mr-2"
              shape="circle"
            />
            {{ data.name }}
          </div>
        </template>
        <template #filter="{ filterModel, filterCallback }">
          <InputText 
            v-model="filterModel.value" 
            type="text" 
            @input="filterCallback()"
            placeholder="Search by name"
          />
        </template>
      </Column>
      
      <Column field="email" header="Email" sortable>
        <template #filter="{ filterModel, filterCallback }">
          <InputText 
            v-model="filterModel.value" 
            type="text" 
            @input="filterCallback()"
            placeholder="Search by email"
          />
        </template>
      </Column>
      
      <Column field="role" header="Role" sortable>
        <template #body="{ data }">
          <Tag 
            :value="data.role" 
            :severity="getRoleSeverity(data.role)"
          />
        </template>
        <template #filter="{ filterModel, filterCallback }">
          <Dropdown 
            v-model="filterModel.value" 
            :options="roleOptions"
            optionLabel="label"
            optionValue="value"
            placeholder="Select Role"
            @change="filterCallback()"
          />
        </template>
      </Column>
      
      <Column field="createdAt" header="Created" sortable dataType="date">
        <template #body="{ data }">
          {{ formatDate(data.createdAt) }}
        </template>
      </Column>
      
      <Column header="Actions">
        <template #body="{ data }">
          <Button 
            icon="pi pi-pencil" 
            class="p-button-rounded p-button-success mr-2"
            @click="editUser(data)"
          />
          <Button 
            icon="pi pi-trash" 
            class="p-button-rounded p-button-warning"
            @click="confirmDeleteUser(data)"
          />
        </template>
      </Column>
    </DataTable>
    
    <!-- User Dialog -->
    <Dialog 
      v-model:visible="userDialog" 
      :style="{ width: '450px' }" 
      header="User Details" 
      :modal="true"
    >
      <div class="field">
        <label for="name">Name</label>
        <InputText 
          id="name" 
          v-model.trim="user.name" 
          required="true" 
          autofocus 
          :class="{ 'p-invalid': submitted && !user.name }"
        />
        <small v-if="submitted && !user.name" class="p-error">Name is required.</small>
      </div>
      
      <div class="field">
        <label for="email">Email</label>
        <InputText 
          id="email" 
          v-model="user.email" 
          required="true"
          :class="{ 'p-invalid': submitted && !user.email }"
        />
        <small v-if="submitted && !user.email" class="p-error">Email is required.</small>
      </div>
      
      <div class="field">
        <label for="role">Role</label>
        <Dropdown 
          id="role" 
          v-model="user.role" 
          :options="roleOptions"
          optionLabel="label"
          optionValue="value"
          placeholder="Select a Role"
        />
      </div>
      
      <template #footer>
        <Button 
          label="Cancel" 
          icon="pi pi-times" 
          class="p-button-text"
          @click="hideDialog"
        />
        <Button 
          label="Save" 
          icon="pi pi-check" 
          class="p-button-text"
          @click="saveUser"
        />
      </template>
    </Dialog>
    
    <Toast />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useToast } from 'primevue/usetoast'
import { FilterMatchMode } from 'primevue/api'

const toast = useToast()

// Data
const users = ref([])
const selectedUsers = ref([])
const user = ref({})
const userDialog = ref(false)
const loading = ref(false)
const submitted = ref(false)

const roleOptions = ref([
  { label: 'User', value: 'user' },
  { label: 'Admin', value: 'admin' },
  { label: 'Moderator', value: 'moderator' }
])

const filters = ref({
  'global': { value: null, matchMode: FilterMatchMode.CONTAINS },
  'name': { value: null, matchMode: FilterMatchMode.CONTAINS },
  'email': { value: null, matchMode: FilterMatchMode.CONTAINS },
  'role': { value: null, matchMode: FilterMatchMode.EQUALS }
})

// Methods
const openNew = () => {
  user.value = { role: 'user' }
  submitted.value = false
  userDialog.value = true
}

const editUser = (userData) => {
  user.value = { ...userData }
  userDialog.value = true
}

const hideDialog = () => {
  userDialog.value = false
  submitted.value = false
}

const saveUser = () => {
  submitted.value = true
  
  if (user.value.name?.trim() && user.value.email?.trim()) {
    if (user.value.id) {
      // Update existing user
      const index = users.value.findIndex(u => u.id === user.value.id)
      users.value[index] = { ...user.value }
      toast.add({
        severity: 'success',
        summary: 'Successful',
        detail: 'User Updated',
        life: 3000
      })
    } else {
      // Create new user
      user.value.id = Date.now()
      user.value.createdAt = new Date()
      users.value.push({ ...user.value })
      toast.add({
        severity: 'success',
        summary: 'Successful',
        detail: 'User Created',
        life: 3000
      })
    }
    
    userDialog.value = false
    user.value = {}
  }
}

const confirmDeleteUser = (userData) => {
  user.value = userData
  // You can use ConfirmDialog here
  if (confirm(`Are you sure you want to delete ${userData.name}?`)) {
    deleteUser()
  }
}

const deleteUser = () => {
  users.value = users.value.filter(u => u.id !== user.value.id)
  user.value = {}
  toast.add({
    severity: 'success',
    summary: 'Successful',
    detail: 'User Deleted',
    life: 3000
  })
}

const confirmDeleteSelected = () => {
  if (confirm(`Delete ${selectedUsers.value.length} selected users?`)) {
    users.value = users.value.filter(u => !selectedUsers.value.includes(u))
    selectedUsers.value = []
    toast.add({
      severity: 'success',
      summary: 'Successful',
      detail: 'Users Deleted',
      life: 3000
    })
  }
}

const getRoleSeverity = (role) => {
  switch (role) {
    case 'admin': return 'danger'
    case 'moderator': return 'warning'
    case 'user': return 'info'
    default: return 'info'
  }
}

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('vi-VN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit'
  })
}

const onRowSelect = (event) => {
  toast.add({
    severity: 'info',
    summary: 'User Selected',
    detail: `${event.data.name} selected`,
    life: 2000
  })
}

const onRowUnselect = (event) => {
  toast.add({
    severity: 'warn',
    summary: 'User Unselected',
    detail: `${event.data.name} unselected`,
    life: 2000
  })
}

// Lifecycle
onMounted(() => {
  // Load sample data
  users.value = [
    {
      id: 1,
      name: 'Nguyen Van A',
      email: 'nguyenvana@example.com',
      role: 'admin',
      createdAt: new Date('2023-01-15'),
      avatar: null
    },
    {
      id: 2,
      name: 'Tran Thi B',
      email: 'tranthib@example.com',
      role: 'user',
      createdAt: new Date('2023-02-20'),
      avatar: null
    },
    {
      id: 3,
      name: 'Le Van C',
      email: 'levanc@example.com',
      role: 'moderator',
      createdAt: new Date('2023-03-10'),
      avatar: null
    }
  ]
})
</script>

<style scoped>
.toolbar {
  margin-bottom: 1rem;
  display: flex;
  gap: 0.5rem;
}

.field {
  margin-bottom: 1rem;
}

.field label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
}

.p-invalid {
  border-color: #e24c4c;
}

.p-error {
  color: #e24c4c;
}
</style>
```

---

## 📚 PHẦN 7: TESTING

### 7.1 Unit Testing với Vitest
```javascript
// tests/components/UserCard.test.js
import { describe, it, expect, beforeEach } from 'vitest'
import { mount } from '@vue/test-utils'
import UserCard from '@/components/UserCard.vue'

describe('UserCard', () => {
  let wrapper
  const mockUser = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    role: 'user'
  }

  beforeEach(() => {
    wrapper = mount(UserCard, {
      props: {
        user: mockUser,
        editable: true
      }
    })
  })

  it('renders user information correctly', () => {
    expect(wrapper.text()).toContain('John Doe')
    expect(wrapper.text()).toContain('john@example.com')
  })

  it('emits update-user event when edit button is clicked', async () => {
    await wrapper.find('button').trigger('click')
    
    expect(wrapper.emitted('update-user')).toBeTruthy()
    expect(wrapper.emitted('update-user')[0][0]).toEqual({
      ...mockUser,
      name: 'Updated Name'
    })
  })

  it('does not show edit button when not editable', () => {
    wrapper = mount(UserCard, {
      props: {
        user: mockUser,
        editable: false
      }
    })
    
    expect(wrapper.find('button').exists()).toBe(false)
  })
})
```

### 7.2 E2E Testing với Cypress
```javascript
// cypress/e2e/user-management.cy.js
describe('User Management', () => {
  beforeEach(() => {
    cy.visit('/users')
  })

  it('should display users table', () => {
    cy.get('[data-cy=users-table]').should('be.visible')
    cy.get('[data-cy=user-row]').should('have.length.greaterThan', 0)
  })

  it('should create new user', () => {
    cy.get('[data-cy=new-user-btn]').click()
    
    cy.get('[data-cy=user-name]').type('Test User')
    cy.get('[data-cy=user-email]').type('test@example.com')
    cy.get('[data-cy=user-role]').select('user')
    
    cy.get('[data-cy=save-user-btn]').click()
    
    cy.contains('User Created').should('be.visible')
    cy.contains('Test User').should('be.visible')
  })

  it('should filter users by name', () => {
    cy.get('[data-cy=name-filter]').type('John')
    
    cy.get('[data-cy=user-row]').should('contain', 'John')
    cy.get('[data-cy=user-row]').should('not.contain', 'Jane')
  })
})
```

---

## 📚 PHẦN 8: PERFORMANCE & OPTIMIZATION

### 8.1 Code Splitting & Lazy Loading
```javascript
// router/index.js - Route-based splitting
const routes = [
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('@/views/Dashboard.vue')
  },
  {
    path: '/users',
    name: 'Users',
    component: () => import('@/views/Users.vue')
  },
  // Chunk naming
  {
    path: '/reports',
    name: 'Reports',
    component: () => import(
      /* webpackChunkName: "reports" */ 
      '@/views/Reports.vue'
    )
  }
]
```

### 8.2 Performance Composables
```javascript
// composables/useVirtualScroll.js
import { ref, computed, onMounted, onUnmounted } from 'vue'

export function useVirtualScroll(items, itemHeight = 50, containerHeight = 400) {
  const scrollTop = ref(0)
  const containerRef = ref(null)
  
  const visibleItems = computed(() => {
    const start = Math.floor(scrollTop.value / itemHeight)
    const visibleCount = Math.ceil(containerHeight / itemHeight)
    const end = Math.min(start + visibleCount + 1, items.value.length)
    
    return items.value.slice(start, end).map((item, index) => ({
      item,
      index: start + index,
      top: (start + index) * itemHeight
    }))
  })
  
  const totalHeight = computed(() => items.value.length * itemHeight)
  
  const onScroll = (event) => {
    scrollTop.value = event.target.scrollTop
  }
  
  onMounted(() => {
    containerRef.value?.addEventListener('scroll', onScroll)
  })
  
  onUnmounted(() => {
    containerRef.value?.removeEventListener('scroll', onScroll)
  })
  
  return {
    containerRef,
    visibleItems,
    totalHeight,
    scrollTop
  }
}
```

### 8.3 Memoization & Caching
```javascript
// composables/useCache.js
import { ref, computed } from 'vue'

export function useCache() {
  const cache = ref(new Map())
  
  const get = (key) => {
    const item = cache.value.get(key)
    if (item && item.expiry > Date.now()) {
      return item.value
    }
    cache.value.delete(key)
    return null
  }
  
  const set = (key, value, ttl = 300000) => { // 5 minutes default
    cache.value.set(key, {
      value,
      expiry: Date.now() + ttl
    })
  }
  
  const clear = () => {
    cache.value.clear()
  }
  
  const size = computed(() => cache.value.size)
  
  return { get, set, clear, size }
}

// Usage with API calls
const { get: getCached, set: setCache } = useCache()

const fetchUsersCached = async () => {
  const cacheKey = 'users'
  let users = getCached(cacheKey)
  
  if (!users) {
    users = await userService.getUsers()
    setCache(cacheKey, users)
  }
  
  return users
}
```

---

## 🔍 KEYWORDS QUAN TRỌNG ĐỂ TÌM HIỂU NÂNG CAO

### Vue.js Core
- **Composition API**: `composables`, `reactive`, `ref`, `computed`, `watch`, `watchEffect`
- **Advanced Reactivity**: `shallowRef`, `triggerRef`, `customRef`, `effectScope`
- **Teleport & Suspense**: async components, error boundaries
- **Custom Directives**: `v-custom`, directive hooks
- **Plugins**: `app.use()`, plugin development

### State Management
- **Pinia Advanced**: `$patch`, `$subscribe`, `$onAction`, store composition
- **State Persistence**: `pinia-plugin-persistedstate`
- **DevTools Integration**: time travel, state inspection

### Routing
- **Route Guards**: `beforeEach`, `beforeResolve`, `afterEach`
- **Meta Fields**: authentication, permissions, breadcrumbs
- **Scroll Behavior**: position, smooth scrolling
- **Nested Routes**: children, params inheritance

### Performance
- **Tree Shaking**: bundle optimization, dead code elimination
- **Virtual Scrolling**: large lists optimization
- **Memoization**: `useMemo`, result caching
- **Code Splitting**: dynamic imports, chunk optimization

### Testing
- **Unit Testing**: `@vue/test-utils`, `vitest`, component testing
- **E2E Testing**: `cypress`, `playwright`, user workflows
- **Component Testing**: props, events, slots testing

### Advanced Topics
- **SSR/SSG**: `Nuxt.js`, server-side rendering
- **PWA**: service workers, offline functionality
- **Micro Frontends**: module federation, component sharing
- **TypeScript**: type safety, interfaces, generics

---

## 🛠️ TOOLS & ECOSYSTEM

### Development Tools
- **Vite**: fast build tool, HMR, plugins
- **Vue DevTools**: debugging, performance profiling
- **ESLint + Prettier**: code quality, formatting
- **Husky**: git hooks, pre-commit checks

### UI Frameworks (alternatives)
- **Vuetify**: Material Design components
- **Quasar**: cross-platform development
- **Element Plus**: enterprise-class components
- **Ant Design Vue**: enterprise UI language

### Utilities
- **VueUse**: collection of Vue composition utilities
- **Lodash**: utility functions for JavaScript
- **Day.js**: lightweight date manipulation
- **Chart.js**: data visualization

---

## 📖 HỌC TIẾP VÀ NÂNG CAO

### Learning Path
1. **Beginner** (4-6 tuần):
   - Vue 3 basics + Composition API
   - Component communication
   - Basic routing và state management

2. **Intermediate** (6-8 tuần):
   - Advanced Composition API patterns
   - Custom composables
   - Performance optimization
   - Testing fundamentals

3. **Advanced** (8+ tuần):
   - SSR với Nuxt.js
   - Micro frontend architecture
   - Custom directive và plugin development
   - Advanced TypeScript integration

### Resources để học
- **Documentation**: Vue.js official docs
- **Courses**: Vue Mastery, Frontend Masters
- **Books**: "Vue.js: Up and Running" by Callum Macrae
- **Blogs**: dev.to/vue, Vue.js community articles
- **YouTube**: Vue.js Amsterdam, VueConf talks

### Thực hành Project Ideas
1. **Todo App**: CRUD operations, local storage
2. **E-commerce**: products, cart, checkout flow
3. **Dashboard**: charts, tables, real-time updates
4. **Blog Platform**: authentication, content management
5. **Real-time Chat**: WebSocket integration

Hãy bắt đầu từ những concept cơ bản và dần dần nâng cao. Thực hành nhiều và xây dựng projects thực tế sẽ giúp bạn hiểu sâu hơn về Vue.js ecosystem!