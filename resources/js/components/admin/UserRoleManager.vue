<script setup>
import { ref, onMounted } from 'vue'
import api from '@/axios'
import { useAuthStore } from '@/stores/auth'

// Reactive array to hold all users
const users = ref([])
// Reactive array to hold all available roles
const allRoles = ref([])

// Object to track selected roles for each user (userId: [roles])
const selectedRoles = ref({}) // { userId: ['admin', 'dispatcher'] }

// Fetch users and roles from the API
const fetchData = async () => {
  // Fetch users and roles in parallel
  const [u, r] = await Promise.all([
    api.get('/admin/users'),
    api.get('/admin/roles/all'),
  ])
  users.value = u.data
  allRoles.value = r.data

  // Initialize selectedRoles for each user based on their current roles
  u.data.forEach(user => {
    selectedRoles.value[user.id] = user.roles.map(role => role.name)
  })
}

// Update roles for a specific user via the API
const updateRoles = async (userId) => {
  await api.post(`/admin/users/${userId}/roles`, {
    roles: selectedRoles.value[userId],
  })
}

// Fetch data when the component is mounted
onMounted(fetchData)
</script>

<template>
  <div class="max-w-5xl mx-auto p-6 space-y-6">
    <h2 class="text-xl font-bold">User Role Assignment</h2>

    <table class="w-full border shadow text-sm">
      <thead class="bg-gray-100">
        <tr>
          <th class="text-left px-3 py-2">Name</th>
          <th class="text-left px-3 py-2">Email</th>
          <th class="text-left px-3 py-2">Roles</th>
          <th class="text-left px-3 py-2">Action</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user.id" class="border-t">
          <td class="px-3 py-2">{{ user.name }}</td>
          <td class="px-3 py-2">{{ user.email }}</td>
          <td class="px-3 py-2">
            <select v-model="selectedRoles[user.id]" multiple class="border rounded w-full p-1">
              <option v-for="role in allRoles" :key="role" :value="role">{{ role }}</option>
            </select>
          </td>
          <td class="px-3 py-2">
            <button
              @click="updateRoles(user.id)"
              class="bg-blue-600 text-white px-3 py-1 rounded"
            >
              Save
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
