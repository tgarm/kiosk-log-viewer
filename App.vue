<template>
  <div id="app">
    <h1>Log Viewer</h1>

    <div class="filters">
      <input v-model="filters.search" placeholder="Search logs" @input="fetchLogs" />
      <input v-model="filters.device_id" placeholder="Filter by Device ID" @input="fetchLogs" />
      <input v-model="filters.app_version" placeholder="Filter by App Version" @input="fetchLogs" />
      <input v-model="filters.app_name" placeholder="Filter by App Name" @input="fetchLogs" />
      <select v-model="filters.log_level" @change="fetchLogs">
        <option value="">All Levels</option>
        <option value="INFO">INFO</option>
        <option value="WARNING">WARNING</option>
        <option value="ERROR">ERROR</option>
      </select>
    </div>

    <table>
      <thead>
        <tr>
          <th @click="sortBy('timestamp')">Timestamp</th>
          <th @click="sortBy('log_level')">Log Level</th>
          <th>Device ID</th>
          <th>App Name</th>
          <th>App Version</th>
          <th>Message</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="log in logs" :key="log.id">
          <td>{{ log.timestamp }}</td>
          <td>{{ log.log_level }}</td>
          <td>{{ log.device_id }}</td>
          <td>{{ log.app_name }}</td>
          <td>{{ log.app_version }}</td>
          <td>{{ log.message }}</td>
          <td><button @click="deleteLog(log.id)">Delete</button></td>
        </tr>
      </tbody>
    </table>

    <div class="pagination">
      <button @click="prevPage" :disabled="filters.offset === 0">Previous</button>
      <button @click="nextPage">Next</button>
    </div>
  </div>
</template>

<script>
import { createClient } from '@supabase/supabase-js'
import axios from 'axios'

const supabaseUrl = 'https://your-supabase-url.supabase.co'
const supabaseKey = 'your-supabase-service-key'
const supabase = createClient(supabaseUrl, supabaseKey)

export default {
  data() {
    return {
      logs: [],
      filters: {
        search: '',
        device_id: '',
        app_version: '',
        app_name: '',
        log_level: '',
        limit: 10,
        offset: 0,
        order_by: 'timestamp',
        order_dir: 'desc'
      }
    }
  },
  methods: {
    async fetchLogs() {
      let query = supabase.from('logs').select('*')

      if (this.filters.search) {
        query = query.ilike('message', `%${this.filters.search}%`)
      }
      if (this.filters.device_id) {
        query = query.eq('device_id', this.filters.device_id)
      }
      if (this.filters.app_version) {
        query = query.eq('app_version', this.filters.app_version)
      }
      if (this.filters.app_name) {
        query = query.eq('app_name', this.filters.app_name)
      }
      if (this.filters.log_level) {
        query = query.eq('log_level', this.filters.log_level)
      }

      query = query.order(this.filters.order_by, { ascending: this.filters.order_dir === 'asc' })
        .range(this.filters.offset, this.filters.offset + this.filters.limit - 1)

      const { data, error } = await query

      if (error) {
        console.error('Error fetching logs:', error)
      } else {
        this.logs = data
      }
    },
    async deleteLog(id) {
      const { error } = await supabase.from('logs').delete().eq('id', id)
      if (error) {
        console.error('Error deleting log:', error)
      } else {
        this.fetchLogs()
      }
    },
    sortBy(field) {
      if (this.filters.order_by === field) {
        this.filters.order_dir = this.filters.order_dir === 'asc' ? 'desc' : 'asc'
      } else {
        this.filters.order_by = field
        this.filters.order_dir = 'asc'
      }
      this.fetchLogs()
    },
    prevPage() {
      if (this.filters.offset > 0) {
        this.filters.offset -= this.filters.limit
        this.fetchLogs()
      }
    },
    nextPage() {
      this.filters.offset += this.filters.limit
      this.fetchLogs()
    }
  },
  mounted() {
    this.fetchLogs()
  }
}
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.filters {
  margin-bottom: 1rem;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 0.5rem;
  border: 1px solid #ddd;
  text-align: left;
}

th {
  cursor: pointer;
}

.pagination {
  margin-top: 1rem;
}
</style>

