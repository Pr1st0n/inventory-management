<template>
  <div class="spending">
    <div class="page-intro">
      <h1 class="page-header__title">{{ t('finance.title') }}</h1>
      <p class="page-header__subtitle">{{ t('finance.description') }}</p>
    </div>

    <div v-if="loading" class="state-message">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="state-message state-message--error">{{ error }}</div>
    <div v-else class="spending__panels">
      <!-- Revenue & Financial KPIs -->
      <div class="grid grid--kpis">
        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">{{ t('finance.totalRevenue') }}</span>
            <span class="stat-tile__icon stat-tile__icon--success">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M4 15l5-5 4 4 7-7" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M15 7h5v5" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </span>
          </div>
          <div class="stat-tile__value">{{ formatCurrency(revenueMetrics.totalRevenue) }}</div>
          <div class="stat-tile__foot">
            <span class="delta delta--up">{{ t('finance.fromOrders', { count: revenueMetrics.orderCount }) }}</span>
          </div>
        </div>

        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">{{ t('finance.totalCosts') }}</span>
            <span class="stat-tile__icon stat-tile__icon--danger">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M12 3v18M8 7.5c0-1.4 1.8-2.5 4-2.5s4 1.1 4 2.5-1.8 2-4 2.5-4 1.1-4 2.5 1.8 2.5 4 2.5 4-1.1 4-2.5" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </span>
          </div>
          <div class="stat-tile__value">{{ formatCurrency(totalCosts) }}</div>
          <div class="stat-tile__foot">
            <span class="stat-tile__since">{{ t('finance.costBreakdown') }}</span>
          </div>
        </div>

        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">{{ t('finance.netProfit') }}</span>
            <span class="stat-tile__icon stat-tile__icon--info">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <rect x="3" y="7" width="18" height="12" rx="2" stroke-width="1.8"/>
                <path d="M3 10h18" stroke-width="1.8"/>
                <path d="M15.5 13h2.5" stroke-width="1.8" stroke-linecap="round"/>
              </svg>
            </span>
          </div>
          <div class="stat-tile__value">{{ formatCurrency(netProfit) }}</div>
          <div class="stat-tile__foot">
            <span class="stat-tile__since">{{ profitMargin }}% {{ t('finance.margin') }}</span>
          </div>
        </div>

        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">{{ t('finance.avgOrderValue') }}</span>
            <span class="stat-tile__icon">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M6 6h13l-1.5 8H8L6 3H3" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                <circle cx="9" cy="19" r="1.4" stroke-width="1.8"/>
                <circle cx="17" cy="19" r="1.4" stroke-width="1.8"/>
              </svg>
            </span>
          </div>
          <div class="stat-tile__value">{{ formatCurrency(revenueMetrics.avgOrderValue) }}</div>
          <div class="stat-tile__foot">
            <span class="stat-tile__since">{{ t('finance.perOrderRevenue') }}</span>
          </div>
        </div>
      </div>

      <!-- Monthly Revenue vs Cost Chart -->
      <div class="card">
        <div class="card__head">
          <h3 class="card__title">{{ t('finance.revenueVsCosts.title') }}</h3>
          <div class="legend">
            <span class="legend__item"><span class="legend__swatch swatch--revenue"></span>{{ t('finance.revenueVsCosts.revenue') }}</span>
            <span class="legend__item"><span class="legend__swatch swatch--cost"></span>{{ t('finance.revenueVsCosts.costs') }}</span>
          </div>
        </div>
        <div class="card__body">
          <div class="bar-chart">
            <div class="y-axis">
              <span>{{ currencySymbol }}{{ maxRevenueValue }}K</span>
              <span>{{ currencySymbol }}{{ Math.round(maxRevenueValue * 0.75) }}K</span>
              <span>{{ currencySymbol }}{{ Math.round(maxRevenueValue * 0.5) }}K</span>
              <span>{{ currencySymbol }}{{ Math.round(maxRevenueValue * 0.25) }}K</span>
              <span>{{ currencySymbol }}0</span>
            </div>
            <div class="chart-area">
              <div v-for="month in monthlyRevenue" :key="month.month" class="bar-group-revenue">
                <div class="revenue-bars">
                  <div class="revenue-bar" :style="{ height: getRevenueBarHeight(month.revenue) + '%' }" :title="`Revenue: ${currencySymbol}${month.revenue.toLocaleString()}`"></div>
                  <div class="cost-bar" :style="{ height: getRevenueBarHeight(month.costs) + '%' }" :title="`Costs: ${currencySymbol}${month.costs.toLocaleString()}`"></div>
                </div>
                <span class="bar-label">{{ translateMonth(month.month) }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Monthly Cost Flow Chart -->
      <div class="card">
        <div class="card__head">
          <h3 class="card__title">{{ t('finance.monthlyCostFlow.title') }}</h3>
          <div class="legend">
            <span class="legend__item"><span class="legend__swatch swatch--procurement"></span>{{ t('finance.monthlyCostFlow.procurement') }}</span>
            <span class="legend__item"><span class="legend__swatch swatch--operational"></span>{{ t('finance.monthlyCostFlow.operational') }}</span>
            <span class="legend__item"><span class="legend__swatch swatch--labor"></span>{{ t('finance.monthlyCostFlow.labor') }}</span>
            <span class="legend__item"><span class="legend__swatch swatch--overhead"></span>{{ t('finance.monthlyCostFlow.overhead') }}</span>
          </div>
        </div>
        <div class="card__body">
          <div class="bar-chart">
            <div class="y-axis">
              <span>{{ currencySymbol }}25K</span>
              <span>{{ currencySymbol }}20K</span>
              <span>{{ currencySymbol }}15K</span>
              <span>{{ currencySymbol }}10K</span>
              <span>{{ currencySymbol }}5K</span>
              <span>{{ currencySymbol }}0</span>
            </div>
            <div class="chart-area">
              <div v-for="month in monthlySpending" :key="month.month" class="bar-group">
                <div class="stacked-bar" @click="showCostDetail(month)">
                  <div class="bar-segment segment--procurement" :style="{ height: getBarHeight(month.procurement) + '%' }" :title="`Procurement: ${currencySymbol}${month.procurement.toLocaleString()}`"></div>
                  <div class="bar-segment segment--operational" :style="{ height: getBarHeight(month.operational) + '%' }" :title="`Operational: ${currencySymbol}${month.operational.toLocaleString()}`"></div>
                  <div class="bar-segment segment--labor" :style="{ height: getBarHeight(month.labor) + '%' }" :title="`Labor: ${currencySymbol}${month.labor.toLocaleString()}`"></div>
                  <div class="bar-segment segment--overhead" :style="{ height: getBarHeight(month.overhead) + '%' }" :title="`Overhead: ${currencySymbol}${month.overhead.toLocaleString()}`"></div>
                </div>
                <span class="bar-label">{{ translateMonth(month.month) }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="grid grid--2">
        <!-- Category Spending Breakdown -->
        <div class="card">
          <div class="card__head">
            <h3 class="card__title">{{ t('finance.categorySpending.title') }}</h3>
          </div>
          <div class="card__body category-list">
            <div v-for="category in categorySpending" :key="category.category" class="category-item">
              <div class="category-info">
                <span class="category-name">{{ translateCategory(category.category) }}</span>
                <span class="category-amount">{{ currencySymbol }}{{ category.amount.toLocaleString() }}</span>
              </div>
              <div class="progress">
                <div class="progress__fill" :style="{ width: category.percentage + '%' }"></div>
              </div>
              <div class="category-meta">
                <span class="percentage">{{ category.percentage }}% {{ t('finance.categorySpending.ofTotal') }}</span>
                <span class="delta" :class="{ 'delta--up': category.change > 0, 'delta--down': category.change < 0 }">
                  {{ category.change > 0 ? '+' : '' }}{{ category.change }}%
                </span>
              </div>
            </div>
          </div>
        </div>

        <!-- Recent Transactions -->
        <div class="card transactions-card">
          <div class="card__head">
            <h3 class="card__title">{{ t('finance.transactions.title') }}</h3>
          </div>
          <div class="transactions-table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>{{ t('finance.transactions.id') }}</th>
                  <th>{{ t('finance.transactions.description') }}</th>
                  <th>{{ t('finance.transactions.vendor') }}</th>
                  <th>{{ t('finance.transactions.date') }}</th>
                  <th class="is-numeric">{{ t('finance.transactions.amount') }}</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="transaction in recentTransactions"
                  :key="transaction.id"
                  class="clickable-row"
                  @click="handleTransactionClick(transaction)"
                >
                  <td><span class="data-table__code">{{ transaction.id.toString().padStart(3, '0') }}</span></td>
                  <td class="data-table__strong">{{ transaction.description }}</td>
                  <td>{{ transaction.vendor }}</td>
                  <td>{{ formatDateShort(transaction.date) }}</td>
                  <td class="is-numeric"><span class="data-table__num">{{ currencySymbol }}{{ transaction.amount.toLocaleString() }}</span></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <CostDetailModal
      :is-open="showCostModal"
      :cost-data="selectedCostData"
      @close="showCostModal = false"
    />
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'
import { formatCurrency as formatCurrencyUtil } from '../utils/currency'
import CostDetailModal from '../components/CostDetailModal.vue'

export default {
  name: 'Spending',
  components: {
    CostDetailModal
  },
  setup() {
    const { t, currentCurrency } = useI18n()
    const loading = ref(true)
    const error = ref(null)
    const allMonthlySpending = ref([])
    const allCategorySpending = ref([])
    const allTransactions = ref([])
    const summaryData = ref({})
    const allOrders = ref([])

    // Modal state
    const showCostModal = ref(false)
    const selectedCostData = ref(null)

    // Use shared filters
    const { selectedPeriod, getCurrentFilters } = useFilters()

    // Monthly spending chart always shows all months (not filtered)
    const monthlySpending = computed(() => {
      return allMonthlySpending.value
    })

    // Filtered monthly spending for summary calculations only
    const filteredMonthlySpending = computed(() => {
      if (selectedPeriod.value === 'all') {
        return allMonthlySpending.value
      }

      // Extract month name from YYYY-MM format
      const monthMap = {
        '01': 'Jan', '02': 'Feb', '03': 'Mar', '04': 'Apr',
        '05': 'May', '06': 'Jun', '07': 'Jul', '08': 'Aug',
        '09': 'Sep', '10': 'Oct', '11': 'Nov', '12': 'Dec'
      }
      const selectedMonth = monthMap[selectedPeriod.value.split('-')[1]]
      return allMonthlySpending.value.filter(m => m.month === selectedMonth)
    })

    const categorySpending = computed(() => {
      return allCategorySpending.value
    })

    const recentTransactions = computed(() => {
      if (selectedPeriod.value === 'all') {
        return allTransactions.value
      }
      // Filter transactions by selected month
      return allTransactions.value.filter(t => {
        const transactionMonth = new Date(t.date).toISOString().slice(0, 7)
        return transactionMonth === selectedPeriod.value
      })
    })

    const summary = computed(() => {
      // Recalculate summary based on filteredMonthlySpending (not the chart data)
      if (filteredMonthlySpending.value.length === 0) {
        return summaryData.value
      }

      const totals = filteredMonthlySpending.value.reduce((acc, month) => ({
        procurement: acc.procurement + month.procurement,
        operational: acc.operational + month.operational,
        labor: acc.labor + month.labor,
        overhead: acc.overhead + month.overhead
      }), { procurement: 0, operational: 0, labor: 0, overhead: 0 })

      return {
        total_procurement_cost: totals.procurement,
        total_operational_cost: totals.operational,
        total_labor_cost: totals.labor,
        total_overhead: totals.overhead,
        procurement_change: summaryData.value.procurement_change || 0,
        operational_change: summaryData.value.operational_change || 0,
        labor_change: summaryData.value.labor_change || 0,
        overhead_change: summaryData.value.overhead_change || 0
      }
    })

    // Filtered orders based on selected period
    const filteredOrders = computed(() => {
      if (selectedPeriod.value === 'all') {
        return allOrders.value
      }

      // Filter orders by selected month
      return allOrders.value.filter(order => {
        const orderMonth = new Date(order.order_date).toISOString().slice(0, 7)
        return orderMonth === selectedPeriod.value
      })
    })

    // Revenue metrics from filtered orders
    const revenueMetrics = computed(() => {
      const totalRevenue = filteredOrders.value.reduce((sum, order) => sum + (order.total_value || 0), 0)
      const orderCount = filteredOrders.value.length
      const avgOrderValue = orderCount > 0 ? totalRevenue / orderCount : 0

      return {
        totalRevenue,
        orderCount,
        avgOrderValue,
        revenueGrowth: 15.3 // Placeholder - could calculate from historical data
      }
    })

    // Total costs from summary
    const totalCosts = computed(() => {
      return summary.value.total_procurement_cost +
             summary.value.total_operational_cost +
             summary.value.total_labor_cost +
             summary.value.total_overhead
    })

    // Net profit
    const netProfit = computed(() => {
      return revenueMetrics.value.totalRevenue - totalCosts.value
    })

    // Profit margin percentage
    const profitMargin = computed(() => {
      if (revenueMetrics.value.totalRevenue === 0) return 0
      return ((netProfit.value / revenueMetrics.value.totalRevenue) * 100).toFixed(1)
    })

    // Monthly revenue data for chart
    const monthlyRevenue = computed(() => {
      const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

      // Initialize all months
      const revenueByMonth = monthNames.map(month => ({
        month,
        revenue: 0,
        costs: 0
      }))

      // Calculate revenue from orders
      allOrders.value.forEach(order => {
        const orderDate = new Date(order.order_date)
        const monthIndex = orderDate.getMonth()
        if (monthIndex >= 0 && monthIndex < 12) {
          revenueByMonth[monthIndex].revenue += order.total_value || 0
        }
      })

      // Add costs from spending data
      allMonthlySpending.value.forEach(spending => {
        const monthIndex = monthNames.indexOf(spending.month)
        if (monthIndex >= 0) {
          revenueByMonth[monthIndex].costs = spending.procurement + spending.operational + spending.labor + spending.overhead
        }
      })

      return revenueByMonth
    })

    // Max value for chart scaling
    const maxRevenueValue = computed(() => {
      const maxRevenue = Math.max(...monthlyRevenue.value.map(m => m.revenue))
      const maxCost = Math.max(...monthlyRevenue.value.map(m => m.costs))
      const max = Math.max(maxRevenue, maxCost)
      return Math.ceil(max / 1000) // Return in K
    })

    const loadData = async () => {
      try {
        loading.value = true
        const [summaryRes, monthlyRes, categoryRes, transactionsRes, ordersRes] = await Promise.all([
          api.getSpendingSummary(),
          api.getMonthlySpending(),
          api.getCategorySpending(),
          api.getTransactions(),
          api.getOrders()
        ])

        summaryData.value = summaryRes
        allMonthlySpending.value = monthlyRes
        allCategorySpending.value = categoryRes
        allTransactions.value = transactionsRes
        allOrders.value = ordersRes
      } catch (err) {
        error.value = 'Failed to load financial data: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch for period filter changes
    watch([selectedPeriod], () => {
      // Data will automatically update via computed properties
    })

    const formatCurrency = (value) => {
      return formatCurrencyUtil(value, currentCurrency.value)
    }

    const currencySymbol = computed(() => {
      return currentCurrency.value === 'JPY' ? '¥' : '$'
    })

    const getBarHeight = (value) => {
      const maxValue = 25000
      return (value / maxValue) * 100
    }

    const getRevenueBarHeight = (value) => {
      const maxValue = maxRevenueValue.value * 1000
      return (value / maxValue) * 100
    }

    const formatDate = (dateString) => {
      return new Date(dateString).toLocaleDateString('en-US', {
        month: 'short',
        day: 'numeric'
      })
    }

    const formatDateShort = (dateString) => {
      const date = new Date(dateString)
      const month = (date.getMonth() + 1).toString().padStart(2, '0')
      const day = date.getDate().toString().padStart(2, '0')
      const year = date.getFullYear().toString().slice(-2)
      return `${month}/${day}/${year}`
    }

    const translateMonth = (month) => {
      const monthMap = {
        'Jan': t('months.jan'),
        'Feb': t('months.feb'),
        'Mar': t('months.mar'),
        'Apr': t('months.apr'),
        'May': t('months.may'),
        'Jun': t('months.jun'),
        'Jul': t('months.jul'),
        'Aug': t('months.aug'),
        'Sep': t('months.sep'),
        'Oct': t('months.oct'),
        'Nov': t('months.nov'),
        'Dec': t('months.dec')
      }
      return monthMap[month] || month
    }

    const translateCategory = (category) => {
      // First try spending categories
      const spendingCategoryMap = {
        'Raw Materials': t('spendingCategories.rawMaterials'),
        'Components': t('spendingCategories.components'),
        'Equipment': t('spendingCategories.equipment'),
        'Consumables': t('spendingCategories.consumables')
      }

      // Then try product categories
      const productCategoryMap = {
        'Circuit Boards': t('categories.circuitBoards'),
        'Sensors': t('categories.sensors'),
        'Actuators': t('categories.actuators'),
        'Controllers': t('categories.controllers'),
        'Power Supplies': t('categories.powerSupplies')
      }

      return spendingCategoryMap[category] || productCategoryMap[category] || category
    }

    const handleTransactionClick = (transaction) => {
      console.log('Transaction clicked:', transaction)
      alert(`Transaction Details:\n\nID: ${transaction.id}\nDescription: ${transaction.description}\nVendor: ${transaction.vendor}\nDate: ${formatDateShort(transaction.date)}\nAmount: $${transaction.amount.toLocaleString()}`)
    }

    const showCostDetail = (monthData) => {
      selectedCostData.value = monthData
      showCostModal.value = true
    }

    onMounted(loadData)

    return {
      t,
      loading,
      error,
      summary,
      monthlySpending,
      categorySpending,
      recentTransactions,
      revenueMetrics,
      totalCosts,
      netProfit,
      profitMargin,
      monthlyRevenue,
      maxRevenueValue,
      formatCurrency,
      currencySymbol,
      getBarHeight,
      getRevenueBarHeight,
      formatDate,
      formatDateShort,
      translateMonth,
      translateCategory,
      handleTransactionClick,
      showCostModal,
      selectedCostData,
      showCostDetail,
      Math
    }
  }
}
</script>

<style scoped>
.spending {
  display: flex;
  flex-direction: column;
  gap: var(--space-7);
}

.page-intro {
  display: flex;
  flex-direction: column;
}

.state-message {
  padding: var(--space-12) var(--space-5);
  text-align: center;
  font-size: var(--text-md);
  color: var(--muted);
}

.state-message--error {
  padding: var(--space-4) var(--space-5);
  text-align: left;
  border: 1px solid var(--danger);
  border-radius: var(--radius-lg);
  background: var(--danger-soft);
  color: var(--danger);
  font-size: var(--text-sm);
  font-weight: var(--fw-medium);
}

.spending__panels {
  display: flex;
  flex-direction: column;
  gap: var(--space-7);
}

/* ---- Legend swatch colors (chart series) --------------------------- */
.legend__swatch.swatch--revenue { background: var(--ink); }
.legend__swatch.swatch--cost { background: var(--danger); }
.legend__swatch.swatch--procurement { background: var(--accent); }
.legend__swatch.swatch--operational { background: var(--muted); }
.legend__swatch.swatch--labor { background: var(--success); }
.legend__swatch.swatch--overhead { background: var(--warning); }

/* ---- Bar charts (hand-built, height driven by inline style) -------- */
.bar-chart {
  display: flex;
  gap: var(--space-6);
  height: 350px;
}

.y-axis {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding-right: var(--space-4);
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  color: var(--faint);
  border-right: 1px solid var(--border);
}

.chart-area {
  flex: 1;
  display: flex;
  align-items: flex-end;
  justify-content: space-around;
  gap: var(--space-2);
}

.bar-group-revenue,
.bar-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
  height: 100%;
}

.revenue-bars {
  width: 100%;
  max-width: 80px;
  display: flex;
  gap: var(--space-2);
  justify-content: center;
  align-items: flex-end;
  height: 100%;
  padding-bottom: var(--space-8);
}

.revenue-bar,
.cost-bar {
  width: 50%;
  max-width: 30px;
  border-radius: var(--radius-xs) var(--radius-xs) 0 0;
  transition: opacity var(--transition-slow), transform var(--transition-slow);
  cursor: pointer;
  min-height: var(--space-1);
}

.revenue-bar { background: var(--ink); }
.cost-bar { background: var(--danger); }

.revenue-bar:hover,
.cost-bar:hover {
  opacity: 0.8;
  transform: scaleY(1.05);
}

.stacked-bar {
  width: 100%;
  max-width: 60px;
  display: flex;
  flex-direction: column-reverse;
  align-items: stretch;
  height: 100%;
  padding-bottom: var(--space-8);
  cursor: pointer;
  transition: opacity var(--transition-base);
}

.stacked-bar:hover { opacity: 0.85; }

.bar-segment {
  width: 100%;
  transition: opacity var(--transition-slow);
  cursor: pointer;
  display: block;
}

.bar-segment:first-child { border-radius: 0 0 var(--radius-xs) var(--radius-xs); }
.bar-segment:last-child { border-radius: var(--radius-xs) var(--radius-xs) 0 0; }

.bar-segment.segment--procurement { background: var(--accent); }
.bar-segment.segment--operational { background: var(--muted); }
.bar-segment.segment--labor { background: var(--success); }
.bar-segment.segment--overhead { background: var(--warning); }

.bar-segment:hover { opacity: 0.8; }

.bar-label {
  margin-top: var(--space-2);
  font-size: var(--text-xs);
  font-weight: var(--fw-semibold);
  color: var(--muted);
}

/* ---- Category spending list ---------------------------------------- */
.category-list {
  display: flex;
  flex-direction: column;
  gap: var(--space-6);
}

.category-item {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}

.category-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.category-name {
  font-weight: var(--fw-semibold);
  color: var(--ink);
}

.category-amount {
  font-family: var(--font-mono);
  font-weight: var(--fw-bold);
  color: var(--accent);
  font-size: var(--text-lg);
  font-variant-numeric: tabular-nums;
}

.progress__fill { transition: width var(--transition-slow); }

.category-meta {
  display: flex;
  justify-content: space-between;
  font-size: var(--text-sm);
}

.percentage { color: var(--muted); }

/* ---- Recent transactions -------------------------------------------- */
.transactions-card {
  display: flex;
  flex-direction: column;
}

.transactions-table-container {
  overflow: auto;
  max-height: 400px;
}

.transactions-table-container .data-table thead th {
  position: sticky;
  top: 0;
  background: var(--surface);
  z-index: var(--z-base);
}

/* Keep the Amount column on one line so it stays fully visible; the container
   scrolls horizontally if the table cannot fit within the card. */
.transactions-table-container .data-table .is-numeric {
  white-space: nowrap;
}

.data-table tbody tr.clickable-row { cursor: pointer; }

.data-table tbody tr.clickable-row:hover td { background: var(--accent-soft); }
</style>
