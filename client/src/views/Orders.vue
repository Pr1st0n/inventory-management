<template>
  <div class="orders">
    <header class="page-header">
      <div class="page-header__titles">
        <h1 class="page-header__title">{{ t('orders.title') }}</h1>
        <p class="page-header__subtitle">{{ t('orders.description') }}</p>
      </div>
    </header>

    <div v-if="loading" class="state-message">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="state-message state-message--error">{{ error }}</div>
    <div v-else>
      <div class="grid grid--kpis">
        <div v-for="statusKey in statusKeys" :key="`kpi-${statusKey}`" class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">{{ t(`status.${statusKey.toLowerCase()}`) }}</span>
            <span :class="['stat-tile__icon', `stat-tile__icon--${getOrderStatusClass(statusKey)}`]">
              <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="7" /></svg>
            </span>
          </div>
          <div class="stat-tile__value">{{ getOrdersByStatus(statusKey).length }}</div>
        </div>
      </div>

      <p class="orders-total">{{ t('orders.allOrders') }} ({{ orders.length }})</p>

      <div
        v-for="statusKey in statusKeys"
        :key="`group-${statusKey}`"
        class="card orders-group"
      >
        <div class="card__head">
          <h3 class="card__title">{{ t(`status.${statusKey.toLowerCase()}`) }}</h3>
          <span :class="['status', `status--${getOrderStatusClass(statusKey)}`]">
            {{ getOrdersByStatus(statusKey).length }}
          </span>
        </div>
        <div class="table-container">
          <table class="data-table orders-table">
            <thead>
              <tr>
                <th class="col-order-number">{{ t('orders.table.orderNumber') }}</th>
                <th class="col-customer">{{ t('orders.table.customer') }}</th>
                <th class="col-items">{{ t('orders.table.items') }}</th>
                <th class="col-date">{{ t('orders.table.orderDate') }}</th>
                <th class="col-date">{{ t('orders.table.expectedDelivery') }}</th>
                <th class="col-value is-numeric">{{ t('orders.table.totalValue') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="order in getOrdersByStatus(statusKey)" :key="order.id">
                <td class="col-order-number">
                  <span class="data-table__code">{{ order.order_number }}</span>
                </td>
                <td class="col-customer">{{ translateCustomerName(order.customer) }}</td>
                <td class="col-items">
                  <details class="items-details">
                    <summary class="items-summary">
                      {{ t('orders.itemsCount', { count: order.items.length }) }}
                    </summary>
                    <div class="items-dropdown">
                      <div v-for="(item, idx) in order.items" :key="idx" class="item-entry">
                        <span class="item-name">{{ translateProductName(item.name) }}</span>
                        <span class="item-meta">{{ t('orders.quantity') }}: {{ item.quantity }} @ {{ currencySymbol }}{{ item.unit_price }}</span>
                      </div>
                    </div>
                  </details>
                </td>
                <td class="col-date">{{ formatDate(order.order_date) }}</td>
                <td class="col-date">{{ formatDate(order.expected_delivery) }}</td>
                <td class="col-value is-numeric">
                  <span class="data-table__num">{{ currencySymbol }}{{ order.total_value.toLocaleString() }}</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Orders',
  setup() {
    const { t, currentCurrency, translateProductName, translateCustomerName } = useI18n()

    const currencySymbol = computed(() => {
      return currentCurrency.value === 'JPY' ? '¥' : '$'
    })
    const loading = ref(true)
    const error = ref(null)
    const orders = ref([])

    // Presentational-only: order of status groups/KPIs rendered in the template.
    const statusKeys = ['Delivered', 'Shipped', 'Processing', 'Backordered']

    // Use shared filters
    const {
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      getCurrentFilters
    } = useFilters()

    const loadOrders = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()
        const fetchedOrders = await api.getOrders(filters)

        // Sort orders by order_date (earliest first)
        orders.value = fetchedOrders.sort((a, b) => {
          const dateA = new Date(a.order_date)
          const dateB = new Date(b.order_date)
          return dateA - dateB
        })
      } catch (err) {
        error.value = 'Failed to load orders: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch for filter changes and reload data
    watch([selectedPeriod, selectedLocation, selectedCategory, selectedStatus], () => {
      loadOrders()
    })

    const getOrdersByStatus = (status) => {
      return orders.value.filter(order => order.status === status)
    }

    const getOrderStatusClass = (status) => {
      const statusMap = {
        'Delivered': 'success',
        'Shipped': 'info',
        'Processing': 'warning',
        'Backordered': 'danger'
      }
      return statusMap[status] || 'info'
    }

    const formatDate = (dateString) => {
      const { currentLocale } = useI18n()
      const locale = currentLocale.value === 'ja' ? 'ja-JP' : 'en-US'
      return new Date(dateString).toLocaleDateString(locale, {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }

    onMounted(loadOrders)

    return {
      t,
      loading,
      error,
      orders,
      statusKeys,
      getOrdersByStatus,
      getOrderStatusClass,
      formatDate,
      currencySymbol,
      translateProductName,
      translateCustomerName
    }
  }
}
</script>

<style scoped>
.page-header {
  position: static;
  background: transparent;
  border-bottom: 0;
  padding: 0;
}

.orders-total {
  margin: 0 0 var(--space-3);
  font-size: var(--text-sm);
  font-weight: var(--fw-medium);
  color: var(--muted);
}

.orders-group + .orders-group {
  margin-top: var(--space-5);
}

.table-container {
  overflow-x: auto;
}

.state-message {
  padding: var(--space-8);
  text-align: center;
  color: var(--muted);
  font-size: var(--text-sm);
}

.state-message--error {
  color: var(--danger);
}

/* Fixed table layout to prevent column shifting */
.orders-table {
  table-layout: fixed;
}

/* Column widths (character-based, not fixed px) */
.col-order-number {
  width: 14ch;
}

.col-customer {
  width: 20ch;
}

.col-items {
  width: 22ch;
}

.col-date {
  width: 15ch;
}

.col-value {
  width: 13ch;
}

/* Items details styling */
.items-details {
  position: relative;
}

.items-summary {
  cursor: pointer;
  color: var(--accent);
  font-weight: var(--fw-medium);
  font-size: var(--text-base);
  list-style: none;
  user-select: none;
  display: inline-block;
}

.items-summary::-webkit-details-marker {
  display: none;
}

.items-summary::before {
  content: '\25B6';
  display: inline-block;
  margin-right: var(--space-1);
  font-size: var(--text-2xs);
  transition: transform var(--transition-fast);
}

.items-details[open] .items-summary::before {
  transform: rotate(90deg);
}

.items-summary:hover {
  color: var(--accent-hover);
  text-decoration: underline;
}

/* Dropdown container */
.items-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  margin-top: var(--space-2);
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-2);
  padding: var(--space-3);
  z-index: var(--z-dropdown);
  min-width: 26ch;
  max-width: 34ch;
}

.item-entry {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-2);
  border-bottom: 1px solid var(--border);
}

.item-entry:last-child {
  border-bottom: none;
}

.item-name {
  font-size: var(--text-sm);
  font-weight: var(--fw-medium);
  color: var(--ink);
}

.item-meta {
  font-size: var(--text-xs);
  color: var(--muted);
}
</style>
