<template>
  <div class="min-h-screen bg-gradient-to-br from-gray-900 via-gray-800 to-gray-900 text-gray-100">
    <NavBar />
    
    <main class="container mx-auto px-4 py-12">
      <h1 class="text-3xl font-bold text-white mb-8">Settings</h1>
      <div class="bg-gray-800 rounded-2xl p-8 shadow-lg border border-gray-700 max-w-2xl mx-auto">
        <!-- Example setting -->
        <div class="mb-6">
          <label class="block text-white font-semibold mb-2">Username</label>
          <input
            v-model="username"
            class="w-full px-4 py-2 rounded-xl bg-gray-900 text-white border border-gray-700 focus:ring-2 focus:ring-pink-500 outline-none"
            type="text"
          />
        </div>
        <!-- Add more settings here -->
        <button class="bg-pink-600 hover:bg-pink-700 text-white font-bold py-2 px-6 rounded-xl shadow">
          Save Changes
        </button>
      </div>
    </main>
  </div>
</template>

<script>
import NavBar from './NavBar.vue'

export default {
  name: 'SettingsPage',
  components: {
    NavBar
  },
  data() {
    return {
      rules: [],
      loading: false,
      saving: false,
      showCreateForm: false,
      editingRule: null,
      ruleForm: {
        label_name: '',
        label_color: '#3B82F6',
        rule_description: '',
        is_active: true
      },
      apiBaseUrl: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000'
    }
  },
  mounted() {
    this.fetchRules()
  },
  methods: {
    async fetchRules() {
      this.loading = true
      try {
        const response = await fetch(`${this.apiBaseUrl}/api/v1/labeling/`)
        if (!response.ok) throw new Error('Failed to fetch rules')
        
        this.rules = await response.json()
      } catch (error) {
        console.error('Error fetching rules:', error)
      } finally {
        this.loading = false
      }
    },
    
    async saveRule() {
      this.saving = true
      try {
        const url = this.editingRule 
          ? `${this.apiBaseUrl}/api/v1/labeling/${this.editingRule.id}`
          : `${this.apiBaseUrl}/api/v1/labeling/`
        
        const method = this.editingRule ? 'PUT' : 'POST'
        
        const response = await fetch(url, {
          method,
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(this.ruleForm)
        })
        
        if (!response.ok) throw new Error('Failed to save rule')
        
        await this.fetchRules()
        this.cancelEdit()
      } catch (error) {
        console.error('Error saving rule:', error)
      } finally {
        this.saving = false
      }
    },
    
    async deleteRule(ruleId) {
      if (!confirm('Are you sure you want to delete this rule? This action cannot be undone.')) {
        return
      }
      
      try {
        const response = await fetch(`${this.apiBaseUrl}/api/v1/labeling/${ruleId}`, {
          method: 'DELETE'
        })
        
        if (!response.ok) throw new Error('Failed to delete rule')
        
        await this.fetchRules()
      } catch (error) {
        console.error('Error deleting rule:', error)
      }
    },
    
    startEdit(rule) {
      this.editingRule = rule
      this.ruleForm = {
        label_name: rule.label_name,
        label_color: rule.label_color,
        rule_description: rule.rule_description,
        is_active: rule.is_active
      }
      this.showCreateForm = false
    },
    
    cancelEdit() {
      this.showCreateForm = false
      this.editingRule = null
      this.ruleForm = {
        label_name: '',
        label_color: '#3B82F6',
        rule_description: '',
        is_active: true
      }
    },
    
    formatDate(dateString) {
      return new Date(dateString).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })
    }
  }
}
</script> 