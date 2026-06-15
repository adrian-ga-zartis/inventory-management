<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') || 'Restocking Planner' }}</h2>
      <p>{{ t('restocking.description') || 'Set your budget and plan restocking orders based on demand forecasts' }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') || 'Loading...' }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <!-- Budget card -->
      <div class="card budget-card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.availableBudget') || 'Available Budget' }}</h3>
        </div>
        <div class="budget-controls">
          <div class="budget-display">${{ budget.toLocaleString() }}</div>
          <input
            type="range"
            min="0"
            max="150000"
            step="5000"
            v-model.number="budget"
          />
          <div class="budget-range-labels">
            <span>$0</span>
            <span>$150,000</span>
          </div>
        </div>
        <div class="budget-bar-section">
          <div class="budget-bar-labels">
            <span>Allocated: ${{ allocatedCost.toLocaleString() }}</span>
            <span>Remaining: ${{ Math.max(0, budget - allocatedCost).toLocaleString() }}</span>
          </div>
          <div class="budget-bar">
            <!-- width is capped at 100% via budgetUsedPercent to prevent overflow -->
            <div
              class="budget-bar-fill"
              :class="{ 'over-budget': overBudget }"
              :style="{ width: budgetUsedPercent + '%' }"
            ></div>
          </div>
        </div>
      </div>

      <!-- Recommended Items card -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">
            {{ t('restocking.recommendedItems') || 'Recommended Restocking' }}
            ({{ withinBudgetItems.length }} {{ t('restocking.items') || 'items' }})
          </h3>
          <span class="badge info">Total: ${{ allocatedCost.toLocaleString() }}</span>
        </div>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') || 'SKU' }}</th>
                <th>{{ t('restocking.table.item') || 'Item' }}</th>
                <th>{{ t('restocking.table.trend') || 'Trend' }}</th>
                <th>{{ t('restocking.table.unitCost') || 'Unit Cost' }}</th>
                <th>{{ t('restocking.table.qty') || 'Qty' }}</th>
                <th>{{ t('restocking.table.subtotal') || 'Subtotal' }}</th>
                <th>{{ t('restocking.table.status') || 'Status' }}</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in sortedItems"
                :key="item.id"
                :class="{ 'row-over-budget': isOverBudgetRow(item) }"
              >
                <td><strong>{{ item.item_sku }}</strong></td>
                <td>{{ item.item_name }}</td>
                <td>
                  <span :class="['badge', trendBadgeClass(item.trend)]">
                    {{ t(`trends.${item.trend}`) || item.trend }}
                  </span>
                </td>
                <td>${{ item.unit_cost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</td>
                <td>
                  <input
                    type="number"
                    class="qty-input"
                    :value="itemQuantities[item.id]"
                    min="0"
                    @input="updateQuantity(item.id, $event.target.value)"
                  />
                </td>
                <td>${{ getItemCost(item).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</td>
                <td>
                  <span v-if="isOverBudgetRow(item)" class="over-budget-label">
                    {{ t('restocking.overBudget') || 'Over budget' }}
                  </span>
                  <span v-else class="within-budget-label">
                    {{ t('restocking.withinBudget') || 'Within budget' }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="card-footer">
          <button
            class="btn-primary"
            @click="placeOrder"
            :disabled="placing || withinBudgetItems.length === 0"
          >
            {{ placing
              ? (t('restocking.placing') || 'Placing...')
              : (t('restocking.placeOrder') || 'Place Order')
            }}
          </button>
          <div v-if="successMessage" class="success-message">{{ successMessage }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t } = useI18n()
    const { getCurrentFilters } = useFilters()

    const loading = ref(true)
    const error = ref(null)
    const demandItems = ref([])

    // Budget slider state — default $50,000
    const budget = ref(50000)

    // Map of item.id → editable quantity; populated after data load
    const itemQuantities = ref({})

    // Submission state
    const placing = ref(false)
    const successMessage = ref('')

    // Filter out "decreasing" trend items, then sort: increasing first, stable second
    const sortedItems = computed(() => {
      return demandItems.value
        .filter(item => item.trend !== 'decreasing')
        .slice()
        .sort((a, b) => {
          const order = { increasing: 0, stable: 1 }
          return (order[a.trend] ?? 2) - (order[b.trend] ?? 2)
        })
    })

    // Cost for a single item based on its current quantity input
    const getItemCost = (item) => {
      const qty = Number(itemQuantities.value[item.id]) || 0
      return qty * item.unit_cost
    }

    // Items whose cumulative running cost (in sorted order) fits within the budget.
    // We walk sortedItems in order, accumulating cost; once we exceed the budget,
    // remaining items are considered "over budget".
    const withinBudgetItems = computed(() => {
      let running = 0
      const result = []
      for (const item of sortedItems.value) {
        const cost = getItemCost(item)
        if (running + cost <= budget.value) {
          running += cost
          result.push(item)
        } else {
          // Stop accumulating once we exceed the budget for this sorted order
          break
        }
      }
      return result
    })

    // Set of IDs that are within budget — used for O(1) row lookups in template
    const withinBudgetIds = computed(() => new Set(withinBudgetItems.value.map(i => i.id)))

    const isOverBudgetRow = (item) => !withinBudgetIds.value.has(item.id)

    // Total cost of all within-budget items
    const allocatedCost = computed(() =>
      withinBudgetItems.value.reduce((sum, item) => sum + getItemCost(item), 0)
    )

    // Percentage of budget consumed, capped at 100 for CSS width binding
    const budgetUsedPercent = computed(() => {
      if (budget.value === 0) return 100
      return Math.min(100, (allocatedCost.value / budget.value) * 100)
    })

    const overBudget = computed(() => allocatedCost.value > budget.value)

    // Trend → badge CSS class mapping
    const trendBadgeClass = (trend) => {
      if (trend === 'increasing') return 'success'
      if (trend === 'stable') return 'info'
      return 'warning'
    }

    const updateQuantity = (id, value) => {
      // Ensure the ref object mutation is reactive by replacing the property
      itemQuantities.value = { ...itemQuantities.value, [id]: Math.max(0, Number(value) || 0) }
    }

    const loadData = async () => {
      loading.value = true
      error.value = null
      try {
        const data = await api.getDemandForecasts()
        demandItems.value = data

        // Initialize quantities to forecasted_demand after data loads
        const quantities = {}
        for (const item of data) {
          quantities[item.id] = item.forecasted_demand
        }
        itemQuantities.value = quantities
      } catch (err) {
        error.value = 'Failed to load demand forecasts: ' + err.message
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const placeOrder = async () => {
      if (placing.value || withinBudgetItems.value.length === 0) return

      placing.value = true
      successMessage.value = ''
      try {
        const items = withinBudgetItems.value.map(item => ({
          sku: item.item_sku,
          name: item.item_name,
          quantity: itemQuantities.value[item.id],
          unit_price: item.unit_cost,
          category: item.category
        }))

        await api.createRestockingOrder({ items })

        successMessage.value = t('restocking.orderSuccess') || 'Order placed successfully.'

        // Reset quantities to forecasted_demand defaults, then clear success message after 3s
        setTimeout(() => {
          const reset = {}
          for (const item of demandItems.value) {
            reset[item.id] = item.forecasted_demand
          }
          itemQuantities.value = reset
          successMessage.value = ''
        }, 3000)
      } catch (err) {
        error.value = 'Failed to place order: ' + err.message
        console.error(err)
      } finally {
        placing.value = false
      }
    }

    onMounted(loadData)

    return {
      t,
      loading,
      error,
      budget,
      itemQuantities,
      placing,
      successMessage,
      sortedItems,
      withinBudgetItems,
      allocatedCost,
      budgetUsedPercent,
      overBudget,
      getItemCost,
      isOverBudgetRow,
      trendBadgeClass,
      updateQuantity,
      placeOrder
    }
  }
}
</script>

<style scoped>
.budget-card {
  margin-bottom: 1.5rem;
}

.budget-controls {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.budget-display {
  font-size: 2rem;
  font-weight: 700;
  color: #0f172a;
}

input[type="range"] {
  width: 100%;
  accent-color: #3b82f6;
  cursor: pointer;
}

.budget-range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.875rem;
  color: #64748b;
}

.budget-bar-section {
  padding: 0 1.5rem 1.5rem;
}

.budget-bar-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.875rem;
  color: #64748b;
  margin-bottom: 0.5rem;
}

.budget-bar {
  height: 8px;
  background: #e2e8f0;
  border-radius: 4px;
  overflow: hidden;
}

.budget-bar-fill {
  height: 100%;
  background: #3b82f6;
  border-radius: 4px;
  transition: width 0.3s ease;
}

.budget-bar-fill.over-budget {
  background: #ef4444;
}

.qty-input {
  width: 80px;
  padding: 0.375rem 0.5rem;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  text-align: right;
}

.row-over-budget {
  opacity: 0.4;
}

.over-budget-label {
  font-size: 0.75rem;
  color: #ef4444;
  font-weight: 500;
}

.within-budget-label {
  font-size: 0.75rem;
  color: #22c55e;
  font-weight: 500;
}

.card-footer {
  padding: 1rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  border-top: 1px solid #e2e8f0;
}

.btn-primary {
  padding: 0.625rem 1.5rem;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-primary:hover:not(:disabled) {
  background: #2563eb;
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.success-message {
  color: #22c55e;
  font-weight: 500;
}
</style>
