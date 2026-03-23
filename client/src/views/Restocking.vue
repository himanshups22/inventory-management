<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <!-- Budget Slider -->
    <div class="card budget-card">
      <div class="card-header">
        <h3 class="card-title">{{ t('restocking.budget') }}</h3>
        <span class="budget-display">{{ currencySymbol }}{{ budget.toLocaleString() }}</span>
      </div>
      <div class="slider-container">
        <span class="slider-min">{{ currencySymbol }}1,000</span>
        <input
          type="range"
          class="budget-slider"
          :min="1000"
          :max="200000"
          :step="1000"
          v-model.number="budget"
          @change="loadRecommendations"
        />
        <span class="slider-max">{{ currencySymbol }}200,000</span>
      </div>
    </div>

    <!-- Recommendations Table -->
    <div class="card">
      <div class="card-header">
        <h3 class="card-title">{{ t('restocking.recommendations') }}</h3>
        <div class="header-actions" v-if="!loadingRecs && recommendations.length > 0">
          <button class="link-btn" @click="selectAll">{{ t('restocking.selectAll') }}</button>
          <span class="separator">|</span>
          <button class="link-btn" @click="deselectAll">{{ t('restocking.deselectAll') }}</button>
        </div>
      </div>

      <div v-if="loadingRecs" class="loading">{{ t('common.loading') }}</div>
      <div v-else-if="errorRecs" class="error">{{ errorRecs }}</div>
      <div v-else-if="recommendations.length === 0" class="empty-state">
        {{ t('restocking.noRecommendations') }}
      </div>
      <div v-else class="table-container">
        <table>
          <thead>
            <tr>
              <th class="col-check"></th>
              <th>{{ t('restocking.table.sku') }}</th>
              <th>{{ t('restocking.table.item') }}</th>
              <th>{{ t('restocking.table.trend') }}</th>
              <th>{{ t('restocking.table.currentDemand') }}</th>
              <th>{{ t('restocking.table.forecastedDemand') }}</th>
              <th>{{ t('restocking.table.quantity') }}</th>
              <th>{{ t('restocking.table.unitCost') }}</th>
              <th>{{ t('restocking.table.totalCost') }}</th>
              <th>{{ t('restocking.table.leadTime') }}</th>
              <th>{{ t('restocking.table.priority') }}</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="item in recommendations"
              :key="item.id"
              :class="{ 'row-selected': selectedIds.has(item.id) }"
              @click="toggleSelection(item)"
            >
              <td class="col-check">
                <input
                  type="checkbox"
                  :checked="selectedIds.has(item.id)"
                  @click.stop="toggleSelection(item)"
                />
              </td>
              <td><strong>{{ item.item_sku }}</strong></td>
              <td>{{ item.item_name }}</td>
              <td>
                <span :class="['badge', item.trend]">{{ item.trend }}</span>
              </td>
              <td>{{ item.current_demand.toLocaleString() }}</td>
              <td><strong>{{ item.forecasted_demand.toLocaleString() }}</strong></td>
              <td>{{ item.quantity_to_order.toLocaleString() }}</td>
              <td>{{ currencySymbol }}{{ item.unit_cost.toFixed(2) }}</td>
              <td><strong>{{ currencySymbol }}{{ item.total_cost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong></td>
              <td>{{ item.lead_time_days }} {{ t('restocking.days') }}</td>
              <td>
                <span :class="['badge', getPriorityClass(item.priority_score)]">
                  {{ item.priority_score }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Summary Bar -->
    <div class="summary-bar" v-if="!loadingRecs && recommendations.length > 0">
      <div class="summary-items">
        <div class="summary-stat">
          <span class="summary-label">{{ t('restocking.selected') }}</span>
          <span class="summary-value">{{ selectedItems.length }}</span>
        </div>
        <div class="summary-stat">
          <span class="summary-label">{{ t('restocking.totalCost') }}</span>
          <span class="summary-value">{{ currencySymbol }}{{ selectedTotalCost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</span>
        </div>
        <div class="summary-stat">
          <span class="summary-label">{{ t('restocking.remaining') }}</span>
          <span class="summary-value" :class="remainingBudget < 0 ? 'over-budget' : ''">
            {{ currencySymbol }}{{ remainingBudget.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}
          </span>
        </div>
      </div>
      <div class="summary-actions">
        <div v-if="orderSuccess" class="success-message">{{ t('restocking.orderSuccess') }}</div>
        <div v-if="orderError" class="error-inline">{{ t('restocking.orderError') }}</div>
        <button
          class="place-order-btn"
          :disabled="selectedItems.length === 0 || placingOrder"
          @click="placeOrder"
        >
          <span v-if="placingOrder">Processing...</span>
          <span v-else>{{ t('restocking.placeOrder') }}</span>
        </button>
      </div>
    </div>

    <!-- Submitted Orders -->
    <div class="card">
      <div class="card-header">
        <h3 class="card-title">{{ t('restocking.submittedOrders') }}</h3>
      </div>

      <div v-if="loadingOrders" class="loading">{{ t('common.loading') }}</div>
      <div v-else-if="errorOrders" class="error">{{ errorOrders }}</div>
      <div v-else-if="submittedOrders.length === 0" class="empty-state">
        {{ t('restocking.noSubmittedOrders') }}
      </div>
      <div v-else class="orders-list">
        <div
          v-for="order in submittedOrders"
          :key="order.id"
          class="order-row"
        >
          <div class="order-summary" @click="toggleOrderExpand(order.id)">
            <div class="order-main">
              <div class="order-number">{{ order.order_number }}</div>
              <div class="order-meta">
                <span>{{ t('restocking.orderDate') }}: {{ formatDate(order.order_date) }}</span>
                <span>{{ t('restocking.expectedDelivery') }}: {{ formatDate(order.expected_delivery) }}</span>
                <span>{{ t('restocking.itemsCount') }}: {{ order.items.length }}</span>
              </div>
            </div>
            <div class="order-right">
              <span class="order-value">{{ currencySymbol }}{{ order.total_value.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</span>
              <span :class="['badge', getOrderStatusClass(order.status)]">{{ order.status }}</span>
              <span class="expand-icon">{{ expandedOrders.has(order.id) ? '&#8722;' : '&#43;' }}</span>
            </div>
          </div>

          <div v-if="expandedOrders.has(order.id)" class="order-items">
            <table>
              <thead>
                <tr>
                  <th>{{ t('restocking.table.sku') }}</th>
                  <th>{{ t('restocking.table.item') }}</th>
                  <th>{{ t('restocking.table.quantity') }}</th>
                  <th>{{ t('restocking.table.unitCost') }}</th>
                  <th>{{ t('restocking.table.totalCost') }}</th>
                  <th>{{ t('restocking.table.leadTime') }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="lineItem in order.items" :key="lineItem.item_sku">
                  <td><strong>{{ lineItem.item_sku }}</strong></td>
                  <td>{{ lineItem.item_name }}</td>
                  <td>{{ lineItem.quantity.toLocaleString() }}</td>
                  <td>{{ currencySymbol }}{{ lineItem.unit_cost.toFixed(2) }}</td>
                  <td><strong>{{ currencySymbol }}{{ lineItem.total_cost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong></td>
                  <td>{{ lineItem.lead_time_days }} {{ t('restocking.days') }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency } = useI18n()

    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')

    // Budget state
    const budget = ref(50000)

    // Recommendations state
    const recommendations = ref([])
    const loadingRecs = ref(false)
    const errorRecs = ref(null)

    // Selection state
    const selectedIds = ref(new Set())

    // Order placement state
    const placingOrder = ref(false)
    const orderSuccess = ref(false)
    const orderError = ref(false)

    // Submitted orders state
    const submittedOrders = ref([])
    const loadingOrders = ref(false)
    const errorOrders = ref(null)

    // Expanded orders state
    const expandedOrders = ref(new Set())

    // Derived: selected recommendation objects
    const selectedItems = computed(() => {
      return recommendations.value.filter(r => selectedIds.value.has(r.id))
    })

    const selectedTotalCost = computed(() => {
      return selectedItems.value.reduce((sum, item) => sum + item.total_cost, 0)
    })

    const remainingBudget = computed(() => {
      return budget.value - selectedTotalCost.value
    })

    const loadRecommendations = async () => {
      loadingRecs.value = true
      errorRecs.value = null
      selectedIds.value = new Set()
      try {
        const data = await api.getRestockingRecommendations(budget.value)
        recommendations.value = data
      } catch (err) {
        errorRecs.value = 'Failed to load recommendations: ' + err.message
        console.error(err)
      } finally {
        loadingRecs.value = false
      }
    }

    const loadSubmittedOrders = async () => {
      loadingOrders.value = true
      errorOrders.value = null
      try {
        const data = await api.getRestockingOrders()
        submittedOrders.value = data
      } catch (err) {
        errorOrders.value = 'Failed to load submitted orders: ' + err.message
        console.error(err)
      } finally {
        loadingOrders.value = false
      }
    }

    const toggleSelection = (item) => {
      const next = new Set(selectedIds.value)
      if (next.has(item.id)) {
        next.delete(item.id)
      } else {
        next.add(item.id)
      }
      selectedIds.value = next
    }

    const selectAll = () => {
      selectedIds.value = new Set(recommendations.value.map(r => r.id))
    }

    const deselectAll = () => {
      selectedIds.value = new Set()
    }

    const placeOrder = async () => {
      if (selectedItems.value.length === 0) return
      placingOrder.value = true
      orderSuccess.value = false
      orderError.value = false
      try {
        const items = selectedItems.value.map(item => ({
          item_sku: item.item_sku,
          item_name: item.item_name,
          quantity: item.quantity_to_order,
          unit_cost: item.unit_cost,
          lead_time_days: item.lead_time_days
        }))
        await api.createRestockingOrder({ items, budget: budget.value })
        orderSuccess.value = true
        deselectAll()
        await loadSubmittedOrders()
        setTimeout(() => { orderSuccess.value = false }, 5000)
      } catch (err) {
        orderError.value = true
        console.error(err)
        setTimeout(() => { orderError.value = false }, 5000)
      } finally {
        placingOrder.value = false
      }
    }

    const toggleOrderExpand = (orderId) => {
      const next = new Set(expandedOrders.value)
      if (next.has(orderId)) {
        next.delete(orderId)
      } else {
        next.add(orderId)
      }
      expandedOrders.value = next
    }

    const getPriorityClass = (score) => {
      if (score >= 400) return 'danger'
      if (score >= 200) return 'warning'
      return 'info'
    }

    const getOrderStatusClass = (status) => {
      const s = (status || '').toLowerCase()
      if (s === 'delivered' || s === 'completed') return 'success'
      if (s === 'processing' || s === 'pending') return 'info'
      if (s === 'cancelled') return 'danger'
      return 'warning'
    }

    const formatDate = (dateStr) => {
      if (!dateStr) return '-'
      const d = new Date(dateStr)
      if (isNaN(d.getTime())) return dateStr
      return d.toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' })
    }

    onMounted(async () => {
      await Promise.all([loadRecommendations(), loadSubmittedOrders()])
    })

    return {
      t,
      currencySymbol,
      budget,
      recommendations,
      loadingRecs,
      errorRecs,
      selectedIds,
      selectedItems,
      selectedTotalCost,
      remainingBudget,
      placingOrder,
      orderSuccess,
      orderError,
      submittedOrders,
      loadingOrders,
      errorOrders,
      expandedOrders,
      loadRecommendations,
      toggleSelection,
      selectAll,
      deselectAll,
      placeOrder,
      toggleOrderExpand,
      getPriorityClass,
      getOrderStatusClass,
      formatDate
    }
  }
}
</script>

<style scoped>
.restocking {
  padding-bottom: 2rem;
}

/* Budget Card */
.budget-card .card-header {
  align-items: center;
}

.budget-display {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.slider-container {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding-top: 0.5rem;
}

.slider-min,
.slider-max {
  font-size: 0.813rem;
  color: #64748b;
  font-weight: 500;
  white-space: nowrap;
  flex-shrink: 0;
}

.budget-slider {
  flex: 1;
  -webkit-appearance: none;
  appearance: none;
  height: 6px;
  border-radius: 4px;
  background: #e2e8f0;
  outline: none;
  cursor: pointer;
  transition: background 0.2s;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #22c55e;
  cursor: pointer;
  border: 2px solid #ffffff;
  box-shadow: 0 1px 4px rgba(34, 197, 94, 0.4);
  transition: box-shadow 0.2s;
}

.budget-slider::-webkit-slider-thumb:hover {
  box-shadow: 0 2px 8px rgba(34, 197, 94, 0.6);
}

.budget-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #22c55e;
  cursor: pointer;
  border: 2px solid #ffffff;
  box-shadow: 0 1px 4px rgba(34, 197, 94, 0.4);
}

/* Header actions */
.header-actions {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.link-btn {
  background: none;
  border: none;
  color: #2563eb;
  font-size: 0.813rem;
  font-weight: 500;
  cursor: pointer;
  padding: 0.25rem 0;
  transition: color 0.15s;
}

.link-btn:hover {
  color: #1d4ed8;
  text-decoration: underline;
}

.separator {
  color: #cbd5e1;
  font-size: 0.813rem;
}

/* Table row selection */
.col-check {
  width: 40px;
}

.row-selected {
  background: #f0fdf4 !important;
}

.row-selected:hover {
  background: #dcfce7 !important;
}

tbody tr {
  cursor: pointer;
}

/* Empty state */
.empty-state {
  padding: 3rem;
  text-align: center;
  color: #64748b;
  font-size: 0.938rem;
}

/* Summary Bar */
.summary-bar {
  background: #0f172a;
  border-radius: 10px;
  padding: 1.25rem 1.5rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.25rem;
  gap: 1rem;
  flex-wrap: wrap;
}

.summary-items {
  display: flex;
  align-items: center;
  gap: 2.5rem;
  flex-wrap: wrap;
}

.summary-stat {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.summary-label {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #94a3b8;
}

.summary-value {
  font-size: 1.25rem;
  font-weight: 700;
  color: #f1f5f9;
  letter-spacing: -0.025em;
}

.summary-value.over-budget {
  color: #f87171;
}

.summary-actions {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.success-message {
  font-size: 0.875rem;
  font-weight: 600;
  color: #4ade80;
}

.error-inline {
  font-size: 0.875rem;
  font-weight: 600;
  color: #f87171;
}

.place-order-btn {
  background: #22c55e;
  color: #ffffff;
  border: none;
  padding: 0.75rem 1.75rem;
  border-radius: 8px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, opacity 0.2s;
  white-space: nowrap;
}

.place-order-btn:hover:not(:disabled) {
  background: #16a34a;
}

.place-order-btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

/* Submitted Orders */
.orders-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.order-row {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  overflow: hidden;
}

.order-summary {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.25rem;
  cursor: pointer;
  transition: background 0.15s;
  background: #f8fafc;
  gap: 1rem;
}

.order-summary:hover {
  background: #f1f5f9;
}

.order-main {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.order-number {
  font-size: 0.938rem;
  font-weight: 700;
  color: #0f172a;
}

.order-meta {
  display: flex;
  gap: 1.5rem;
  flex-wrap: wrap;
}

.order-meta span {
  font-size: 0.813rem;
  color: #64748b;
}

.order-right {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-shrink: 0;
}

.order-value {
  font-size: 1rem;
  font-weight: 700;
  color: #0f172a;
}

.expand-icon {
  font-size: 1.25rem;
  color: #64748b;
  line-height: 1;
  width: 20px;
  text-align: center;
}

.order-items {
  border-top: 1px solid #e2e8f0;
  background: #ffffff;
  padding: 0 0.25rem 0.5rem;
}
</style>
