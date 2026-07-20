<template>
  <div class="dashboard">
    <div v-if="loading" class="state-msg">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="state-msg state-msg--error">{{ error }}</div>
    <div v-else class="dash-body">
      <h2 class="page-header__title">{{ t('dashboard.title') }}</h2>

      <!-- Key Performance Indicators -->
      <section>
        <h3 class="section-title">{{ t('dashboard.kpi.title') }}</h3>
        <div class="grid grid--kpis rise">
          <div class="stat-tile">
            <div class="stat-tile__head">
              <span class="stat-tile__label">{{ t('dashboard.kpi.inventoryTurnover') }}</span>
              <span class="stat-tile__icon stat-tile__icon--info">
                <svg viewBox="0 0 24 24"><path d="M23 4v6h-6"/><path d="M20.49 15a9 9 0 1 1-2.12-9.36L23 10"/></svg>
              </span>
            </div>
            <div class="stat-tile__value">4.2</div>
            <div class="progress"><div class="progress__fill" style="width: 93.33%"></div></div>
            <div class="stat-tile__foot">
              <span class="stat-tile__since">{{ t('dashboard.kpi.goal') }}: 4.5 (-6.67%)</span>
            </div>
          </div>

          <div class="stat-tile">
            <div class="stat-tile__head">
              <span class="stat-tile__label">{{ t('dashboard.kpi.ordersFulfilled') }}</span>
              <span class="stat-tile__icon stat-tile__icon--success">
                <svg viewBox="0 0 24 24"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
              </span>
            </div>
            <div class="stat-tile__value">{{ ordersData.fulfilled }}</div>
            <div class="progress"><div class="progress__fill" :style="{ width: calculatePercentage(ordersData.fulfilled, ordersData.goal) + '%' }"></div></div>
            <div class="stat-tile__foot">
              <span class="stat-tile__since">{{ t('dashboard.kpi.goal') }}: {{ ordersData.goal }} ({{ calculatePercentage(ordersData.fulfilled, ordersData.goal) }}%)</span>
            </div>
          </div>

          <div class="stat-tile">
            <div class="stat-tile__head">
              <span class="stat-tile__label">{{ t('dashboard.kpi.orderFillRate') }}</span>
              <span class="stat-tile__icon stat-tile__icon--info">
                <svg viewBox="0 0 24 24"><path d="M3 3v18h18"/><path d="M18 17V9M13 17V5M8 17v-3"/></svg>
              </span>
            </div>
            <div class="stat-tile__value">{{ fillRate }}%</div>
            <div class="progress"><div class="progress__fill" :style="{ width: (fillRate / 95 * 100) + '%' }"></div></div>
            <div class="stat-tile__foot">
              <span class="stat-tile__since">{{ t('dashboard.kpi.goal') }}: 95% ({{ fillRate - 95 > 0 ? '+' : '' }}{{ (fillRate - 95).toFixed(2) }}%)</span>
            </div>
          </div>

          <div class="stat-tile">
            <div class="stat-tile__head">
              <span class="stat-tile__label">{{ t(selectedPeriod === 'all' ? 'dashboard.kpi.revenueYTD' : 'dashboard.kpi.revenueMTD') }}</span>
              <span class="stat-tile__icon stat-tile__icon--success">
                <svg viewBox="0 0 24 24"><path d="M23 6l-9.5 9.5-5-5L1 18"/><path d="M17 6h6v6"/></svg>
              </span>
            </div>
            <div class="stat-tile__value">{{ formatCurrency(Math.round(summary.total_orders_value), selectedCurrency) }}</div>
            <div class="progress"><div class="progress__fill" :style="{ width: Math.min((summary.total_orders_value / revenueGoal * 100), 100) + '%' }"></div></div>
            <div class="stat-tile__foot">
              <span class="stat-tile__since">{{ t('dashboard.kpi.goal') }}: {{ formatCurrency(revenueGoal, selectedCurrency) }} ({{ summary.total_orders_value > revenueGoal ? '+' : '' }}{{ ((summary.total_orders_value / revenueGoal - 1) * 100).toFixed(1) }}%)</span>
            </div>
          </div>

          <div class="stat-tile">
            <div class="stat-tile__head">
              <span class="stat-tile__label">{{ t('dashboard.kpi.avgProcessingTime') }}</span>
              <span class="stat-tile__icon stat-tile__icon--warning">
                <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 2"/></svg>
              </span>
            </div>
            <div class="stat-tile__value">2.8</div>
            <div class="progress"><div class="progress__fill" style="width: 93.33%"></div></div>
            <div class="stat-tile__foot">
              <span class="stat-tile__since">{{ t('dashboard.kpi.goal') }}: 3.0 (-6.67%)</span>
            </div>
          </div>
        </div>
      </section>

      <!-- Summary / Charts -->
      <section>
        <h3 class="section-title">{{ t('dashboard.summary.title') }}</h3>
        <div class="grid grid--wide-narrow">
          <!-- Order Health Dashboard -->
          <div class="card">
            <div class="card__head">
              <h3 class="card__title">{{ t('dashboard.orderHealth.title') }}</h3>
              <span class="card__meta">{{ orderHealthMetrics.totalOrders }}</span>
            </div>
            <div class="card__body">
              <div class="order-health">
                <!-- Left: Donut Chart -->
                <div class="order-health__chart">
                  <div class="donut-center">
                    <svg width="184" height="184" viewBox="0 0 200 200" class="donut">
                      <circle class="donut__track" cx="100" cy="100" r="65" fill="none" stroke-width="25"/>
                      <circle class="donut__seg donut__seg--delivered" cx="100" cy="100" r="65" fill="none" stroke-width="25"
                        :stroke-dasharray="`${getCircleSegment(statusData.delivered)} 408`"
                        stroke-dashoffset="0" transform="rotate(-90 100 100)"/>
                      <circle class="donut__seg donut__seg--shipped" cx="100" cy="100" r="65" fill="none" stroke-width="25"
                        :stroke-dasharray="`${getCircleSegment(statusData.shipped)} 408`"
                        :stroke-dashoffset="`-${getCircleSegment(statusData.delivered)}`"
                        transform="rotate(-90 100 100)"/>
                      <circle class="donut__seg donut__seg--processing" cx="100" cy="100" r="65" fill="none" stroke-width="25"
                        :stroke-dasharray="`${getCircleSegment(statusData.processing)} 408`"
                        :stroke-dashoffset="`-${getCircleSegment(statusData.delivered) + getCircleSegment(statusData.shipped)}`"
                        transform="rotate(-90 100 100)"/>
                      <circle class="donut__seg donut__seg--backordered" cx="100" cy="100" r="65" fill="none" stroke-width="25"
                        :stroke-dasharray="`${getCircleSegment(statusData.backordered)} 408`"
                        :stroke-dashoffset="`-${getCircleSegment(statusData.delivered) + getCircleSegment(statusData.shipped) + getCircleSegment(statusData.processing)}`"
                        transform="rotate(-90 100 100)"/>
                    </svg>
                    <div class="donut-center__value">
                      <div>
                        <b>{{ orderHealthMetrics.totalOrders }}</b>
                        <span>{{ t('dashboard.orderHealth.total') }}</span>
                      </div>
                    </div>
                  </div>
                  <div class="legend">
                    <span class="legend__item"><span class="legend__swatch legend__swatch--delivered"></span>{{ t('status.delivered') }}</span>
                    <span class="legend__item"><span class="legend__swatch legend__swatch--shipped"></span>{{ t('status.shipped') }}</span>
                    <span class="legend__item"><span class="legend__swatch legend__swatch--processing"></span>{{ t('status.processing') }}</span>
                    <span class="legend__item"><span class="legend__swatch legend__swatch--backordered"></span>{{ t('status.backordered') }}</span>
                  </div>
                </div>

                <!-- Right: Health Metrics -->
                <div class="order-health__metrics">
                  <div class="metric">
                    <span class="metric__label">{{ t('dashboard.orderHealth.revenue') }}</span>
                    <span class="metric__value">{{ formatCurrency(orderHealthMetrics.totalValue, selectedCurrency) }}</span>
                  </div>
                  <div class="metric">
                    <span class="metric__label">{{ t('dashboard.orderHealth.avgOrderValue') }}</span>
                    <span class="metric__value">{{ formatCurrency(orderHealthMetrics.avgOrderValue, selectedCurrency) }}</span>
                  </div>
                  <div class="metric">
                    <span class="metric__label">{{ t('dashboard.orderHealth.onTimeRate') }}</span>
                    <span class="metric__value" :class="{ 'metric__value--good': orderHealthMetrics.onTimeRate >= 90, 'metric__value--warning': orderHealthMetrics.onTimeRate < 90 && orderHealthMetrics.onTimeRate >= 75, 'metric__value--bad': orderHealthMetrics.onTimeRate < 75 }">
                      {{ orderHealthMetrics.onTimeRate.toFixed(1) }}%
                    </span>
                  </div>
                  <div class="metric">
                    <span class="metric__label">{{ t('dashboard.orderHealth.avgFulfillmentDays') }}</span>
                    <span class="metric__value">{{ orderHealthMetrics.avgFulfillmentDays.toFixed(1) }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Inventory by Category -->
          <div class="card">
            <div class="card__head">
              <h3 class="card__title">{{ t('dashboard.inventoryValue.title') }}</h3>
            </div>
            <div class="card__body">
              <div class="bars" v-if="categoryData.length > 0">
                <div v-for="cat in categoryData" :key="cat.name" class="bar-row">
                  <div class="bar-row__meta">
                    <b>{{ translateCategory(cat.name) }}</b>
                    <span>{{ selectedCurrency === 'JPY' ? formatCurrency(cat.value, selectedCurrency) : `$${(cat.value / 1000).toFixed(1)}K` }}</span>
                  </div>
                  <div class="progress"><div class="progress__fill" :style="{ width: (cat.value / maxCategoryValue * 100) + '%' }"></div></div>
                </div>
              </div>
              <div v-else class="no-data">{{ t('dashboard.inventoryShortages.noData') }}</div>
            </div>
          </div>
        </div>
      </section>

      <!-- Inventory Shortages -->
      <div class="card">
        <div class="card__head">
          <h3 class="card__title">{{ t('dashboard.inventoryShortages.title') }} ({{ backlogItems.length }})</h3>
        </div>
        <div v-if="backlogItems.length === 0" class="no-shortage">
          <svg class="no-shortage__icon" width="44" height="44" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
          </svg>
          <p class="no-shortage__text">{{ t('dashboard.inventoryShortages.noShortages') }}</p>
        </div>
        <div v-else class="table-scroll">
          <table class="data-table">
            <thead>
              <tr>
                <th>{{ t('dashboard.inventoryShortages.orderId') }}</th>
                <th>{{ t('dashboard.inventoryShortages.sku') }}</th>
                <th>{{ t('dashboard.inventoryShortages.itemName') }}</th>
                <th class="is-numeric">{{ t('dashboard.inventoryShortages.quantityNeeded') }}</th>
                <th class="is-numeric">{{ t('dashboard.inventoryShortages.quantityAvailable') }}</th>
                <th>{{ t('dashboard.inventoryShortages.shortage') }}</th>
                <th>{{ t('dashboard.inventoryShortages.daysDelayed') }}</th>
                <th>{{ t('dashboard.inventoryShortages.priority') }}</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in backlogItems"
                :key="item.id"
              >
                <td class="cell-link" @click="showBacklogDetail(item)"><span class="data-table__code">{{ item.order_id }}</span></td>
                <td class="cell-link" @click="showBacklogDetail(item)"><span class="data-table__code">{{ item.item_sku }}</span></td>
                <td class="cell-link" @click="showBacklogDetail(item)">{{ translateProductName(item.item_name) }}</td>
                <td class="cell-link is-numeric" @click="showBacklogDetail(item)"><span class="data-table__num">{{ item.quantity_needed }}</span></td>
                <td class="cell-link is-numeric" @click="showBacklogDetail(item)"><span class="data-table__num">{{ item.quantity_available }}</span></td>
                <td class="cell-link" @click="showBacklogDetail(item)">
                  <span class="badge badge--danger">
                    {{ Math.abs(item.quantity_needed - item.quantity_available) }} {{ t('dashboard.inventoryShortages.unitsShort') }}
                  </span>
                </td>
                <td class="cell-link" @click="showBacklogDetail(item)">
                  <span :class="item.days_delayed > 7 ? 'text-danger' : 'text-warning'">
                    {{ item.days_delayed }} {{ t('dashboard.inventoryShortages.days') }}
                  </span>
                </td>
                <td class="cell-link" @click="showBacklogDetail(item)">
                  <span class="badge" :class="{
                    'badge--danger': item.priority.toLowerCase() === 'high',
                    'badge--warning': item.priority.toLowerCase() === 'medium',
                    'badge--success': item.priority.toLowerCase() === 'low'
                  }">
                    {{ translatePriority(item.priority) }}
                  </span>
                </td>
                <td>
                  <button
                    v-if="!item.purchase_order_id"
                    @click.stop="openPOModal(item)"
                    class="btn btn-primary btn-sm"
                  >
                    Create PO
                  </button>
                  <button
                    v-else
                    @click.stop="viewPO(item)"
                    class="btn btn-ghost btn-sm"
                  >
                    View PO
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Top Products Table -->
      <div class="card">
        <div class="card__head">
          <h3 class="card__title">{{ t('dashboard.topProducts.title') }}</h3>
        </div>
        <div class="table-scroll">
          <table class="data-table">
            <thead>
              <tr>
                <th>{{ t('dashboard.topProducts.product') }}</th>
                <th>{{ t('dashboard.topProducts.sku') }}</th>
                <th>{{ t('dashboard.topProducts.category') }}</th>
                <th class="is-numeric">{{ t('dashboard.topProducts.unitsOrdered') }}</th>
                <th class="is-numeric">{{ t('dashboard.topProducts.revenue') }}</th>
                <th>{{ t('dashboard.topProducts.firstOrder') }}</th>
                <th>{{ t('dashboard.topProducts.stockStatus') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in topProducts"
                :key="item.sku"
                class="row-link"
                @click="showProductDetail(item)"
              >
                <td><span class="data-table__strong">{{ translateProductName(item.name) }}</span></td>
                <td><span class="data-table__code">{{ item.sku }}</span></td>
                <td>{{ translateCategory(item.category) }}</td>
                <td class="is-numeric"><span class="data-table__num">{{ item.unitsOrdered }}</span></td>
                <td class="is-numeric"><span class="data-table__num">{{ formatCurrency(item.revenue, selectedCurrency) }}</span></td>
                <td>{{ formatDate(item.firstOrderDate) }}</td>
                <td>
                  <span :class="['status', 'status--' + getStockBadge(item.stockLevel)]">
                    {{ translateStockLevel(item.stockLevel) }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <ProductDetailModal
      :is-open="showProductModal"
      :product="selectedProduct"
      @close="showProductModal = false"
    />

    <BacklogDetailModal
      :is-open="showBacklogModal"
      :backlog-item="selectedBacklogItem"
      @close="showBacklogModal = false"
    />

    <PurchaseOrderModal
      :is-open="showPOModal"
      :backlog-item="selectedBacklogForPO"
      :mode="poModalMode"
      @close="showPOModal = false"
      @po-created="handlePOCreated"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed, watch } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'
import { formatCurrency } from '../utils/currency'
import ProductDetailModal from '../components/ProductDetailModal.vue'
import BacklogDetailModal from '../components/BacklogDetailModal.vue'

export default {
  name: 'Dashboard',
  components: {
    ProductDetailModal,
    BacklogDetailModal,
  },
  setup() {
    const { t, currentCurrency, translateProductName, translateWarehouse } = useI18n()
    const loading = ref(true)
    const error = ref(null)
    const summary = ref({})
    const allOrders = ref([])
    const inventoryItems = ref([])

    // Modal state
    const showProductModal = ref(false)
    const selectedProduct = ref(null)
    const showBacklogModal = ref(false)
    const selectedBacklogItem = ref(null)
    const showPOModal = ref(false)
    const selectedBacklogForPO = ref(null)
    const poModalMode = ref('create')

    // Use shared filters
    const {
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      getCurrentFilters
    } = useFilters()

    const ordersData = ref({ fulfilled: 187, goal: 200 })
    const fillRate = ref(96.8)

    const revenueGoal = computed(() => {
      // $800K per month, so if looking at all months (12 months), goal is 12 * 800K = 9.6M
      const monthlyGoal = 800000
      if (selectedPeriod.value === 'all') {
        return monthlyGoal * 12 // $9,600,000 for the full year
      }
      return monthlyGoal // $800,000 for a single month
    })

    const revenueGoalDisplay = computed(() => {
      if (revenueGoal.value >= 1000000) {
        return `$${(revenueGoal.value / 1000000).toFixed(1)}M`
      }
      return `$${(revenueGoal.value / 1000).toFixed(0)}K`
    })

    const statusData = computed(() => {
      const counts = { delivered: 0, shipped: 0, processing: 0, backordered: 0 }
      allOrders.value.forEach(order => {
        const status = order.status.toLowerCase()
        if (counts[status] !== undefined) counts[status]++
      })
      return counts
    })

    const orderHealthMetrics = computed(() => {
      const totalOrders = allOrders.value.length
      const totalValue = allOrders.value.reduce((sum, order) => sum + (order.total_value || 0), 0)
      const avgOrderValue = totalOrders > 0 ? totalValue / totalOrders : 0

      // Calculate on-time delivery rate (delivered orders that arrived on or before expected date)
      const deliveredOrders = allOrders.value.filter(o => o.status.toLowerCase() === 'delivered')
      const onTimeDeliveries = deliveredOrders.filter(o => {
        if (o.actual_delivery && o.expected_delivery) {
          return new Date(o.actual_delivery) <= new Date(o.expected_delivery)
        }
        return false
      }).length
      const onTimeRate = deliveredOrders.length > 0 ? (onTimeDeliveries / deliveredOrders.length) * 100 : 0

      // Calculate average fulfillment speed (days from order to delivery for delivered orders)
      let totalDays = 0
      let countWithDates = 0
      deliveredOrders.forEach(o => {
        if (o.order_date && o.actual_delivery) {
          const orderDate = new Date(o.order_date)
          const deliveryDate = new Date(o.actual_delivery)
          const days = Math.round((deliveryDate - orderDate) / (1000 * 60 * 60 * 24))
          totalDays += days
          countWithDates++
        }
      })
      const avgFulfillmentDays = countWithDates > 0 ? totalDays / countWithDates : 0

      return {
        totalOrders,
        totalValue,
        avgOrderValue,
        onTimeRate,
        avgFulfillmentDays
      }
    })

    const categoryData = computed(() => {
      // Group inventory by category and calculate values
      // Filter inventory items to only include those with orders in the selected period
      const categoryMap = {}

      // Use a single neutral slate/gray color for all categories
      const singleColor = '#64748b' // Neutral slate gray color

      // Get SKUs from orders in the filtered time period
      const orderedSkus = new Set()
      allOrders.value.forEach(order => {
        if (order.items) {
          order.items.forEach(item => {
            orderedSkus.add(item.sku)
          })
        }
      })

      // Only include inventory items that have orders in the selected period
      // If no period is selected (all), include all inventory items
      const itemsToInclude = selectedPeriod.value === 'all'
        ? inventoryItems.value
        : inventoryItems.value.filter(item => orderedSkus.has(item.sku))

      itemsToInclude.forEach(item => {
        const cat = item.category.toLowerCase()
        if (!categoryMap[cat]) {
          categoryMap[cat] = {
            name: item.category,
            value: 0,
            color: singleColor,
            category: cat,
            count: 0
          }
        }
        categoryMap[cat].value += item.quantity_on_hand * item.unit_cost
        categoryMap[cat].count += 1
      })

      return Object.values(categoryMap)
    })

    const maxCategoryValue = computed(() => {
      if (categoryData.value.length === 0) return 1
      return Math.max(...categoryData.value.map(c => c.value))
    })

    const orderTrendData = computed(() => {
      // Group orders by month from the actual data
      const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

      // Initialize all months with 0 orders
      const monthMap = {}
      monthNames.forEach(month => {
        monthMap[month] = { month, orders: 0 }
      })

      // Count orders for each month
      if (Array.isArray(allOrders.value)) {
        allOrders.value.forEach(order => {
          if (order && order.order_date) {
            const date = new Date(order.order_date)
            const monthIndex = date.getMonth()
            // Check if monthIndex is valid (0-11)
            if (!isNaN(monthIndex) && monthIndex >= 0 && monthIndex <= 11) {
              const monthName = monthNames[monthIndex]
              monthMap[monthName].orders++
            }
          }
        })
      }

      // Return all months in order
      return monthNames.map(month => monthMap[month])
    })

    const maxOrderCount = computed(() => {
      if (orderTrendData.value.length === 0) return 10
      const max = Math.max(...orderTrendData.value.map(d => d.orders))
      // Round up to nearest 10 for cleaner axis, minimum 10
      return Math.max(10, Math.ceil(max / 10) * 10)
    })

    const topProducts = computed(() => {
      // Calculate top products from filtered order data
      const productMap = {}

      // allOrders is already filtered by API based on: month, warehouse, category, status
      allOrders.value.forEach(order => {
        if (order.items) {
          order.items.forEach(item => {
            const sku = item.sku

            // Find matching inventory item to get full product details
            // Note: inventoryItems is also filtered by API based on: warehouse, category
            const invItem = inventoryItems.value.find(i => i.sku === sku)

            // Skip products that don't match current inventory filters
            // (e.g., if filtering by warehouse A, don't show products from warehouse B)
            if (!invItem && (selectedLocation.value !== 'all' || selectedCategory.value !== 'all')) {
              return // Skip this product as it doesn't match inventory filters
            }

            if (!productMap[sku]) {
              productMap[sku] = {
                name: item.name,
                sku: sku,
                category: invItem?.category || 'Unknown',
                warehouse: invItem?.warehouse || 'Unknown',
                unitsOrdered: 0,
                revenue: 0,
                stockLevel: invItem ? (invItem.quantity_on_hand > invItem.reorder_point ? 'In Stock' : 'Low Stock') : 'Unknown',
                firstOrderDate: order.order_date
              }
            } else {
              // Update to EARLIEST order date (to show January at top when selecting All Months)
              if (order.order_date && (!productMap[sku].firstOrderDate || order.order_date < productMap[sku].firstOrderDate)) {
                productMap[sku].firstOrderDate = order.order_date
              }
            }
            productMap[sku].unitsOrdered += item.quantity
            productMap[sku].revenue += item.quantity * item.unit_price
          })
        }
      })

      // Convert to array, sort by first order date (earliest first = January at top), then by revenue, and take top 12
      return Object.values(productMap)
        .sort((a, b) => {
          // Sort by first order date (earliest first)
          // This ensures products first ordered in January appear before those first ordered in December
          const dateA = new Date(a.firstOrderDate || '9999-12-31')
          const dateB = new Date(b.firstOrderDate || '9999-12-31')
          if (dateA.getTime() !== dateB.getTime()) {
            return dateA.getTime() - dateB.getTime() // Earlier dates come first
          }
          // If dates are equal, sort by revenue (highest first)
          return b.revenue - a.revenue
        })
        .slice(0, 12)
    })

    const allBacklogItems = ref([])

    // Filter backlog based on inventory filters
    const backlogItems = computed(() => {
      if (selectedLocation.value === 'all' && selectedCategory.value === 'all') {
        return allBacklogItems.value
      }

      // Get SKUs of items that match the filters
      const validSkus = new Set(inventoryItems.value.map(item => item.sku))
      return allBacklogItems.value.filter(b => validSkus.has(b.item_sku))
    })

    const loadData = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()

        const [summaryData, ordersData, inventoryData, backlogData] = await Promise.all([
          api.getDashboardSummary(filters),
          api.getOrders(filters),
          api.getInventory(filters),
          api.getBacklog()
        ])

        summary.value = summaryData
        allOrders.value = ordersData
        inventoryItems.value = inventoryData
        allBacklogItems.value = backlogData
      } catch (err) {
        error.value = 'Failed to load dashboard data: ' + err.message
      } finally {
        loading.value = false
      }
    }

    const calculatePercentage = (value, goal) => {
      return ((value / goal) * 100).toFixed(2)
    }

    // Compute total orders once for efficiency
    const totalOrders = computed(() => {
      return statusData.value.delivered + statusData.value.shipped +
             statusData.value.processing + statusData.value.backordered
    })

    const getCircleSegment = (value) => {
      return totalOrders.value > 0 ? (value / totalOrders.value) * 440 : 0
    }

    const getStockBadge = (level) => {
      if (level === 'In Stock') return 'success'
      if (level === 'Low Stock') return 'warning'
      return 'danger'
    }

    const translateCategory = (category) => {
      const categoryMap = {
        'Circuit Boards': t('categories.circuitBoards'),
        'Sensors': t('categories.sensors'),
        'Actuators': t('categories.actuators'),
        'Controllers': t('categories.controllers'),
        'Power Supplies': t('categories.powerSupplies')
      }
      return categoryMap[category] || category
    }

    const translateStockLevel = (stockLevel) => {
      const stockMap = {
        'In Stock': t('status.inStock'),
        'Low Stock': t('status.lowStock')
      }
      return stockMap[stockLevel] || stockLevel
    }

    const translatePriority = (priority) => {
      const priorityMap = {
        'high': t('priority.high'),
        'medium': t('priority.medium'),
        'low': t('priority.low'),
        'High': t('priority.high'),
        'Medium': t('priority.medium'),
        'Low': t('priority.low')
      }
      return priorityMap[priority] || priority
    }

    const formatDate = (dateString) => {
      if (!dateString) return '-'
      const { currentLocale } = useI18n()
      const locale = currentLocale.value === 'ja' ? 'ja-JP' : 'en-US'
      const date = new Date(dateString)
      return date.toLocaleDateString(locale, { month: 'short', day: 'numeric', year: 'numeric' })
    }

    const showProductDetail = (product) => {
      selectedProduct.value = product
      showProductModal.value = true
    }

    const showBacklogDetail = (item) => {
      selectedBacklogItem.value = item
      showBacklogModal.value = true
    }

    const openPOModal = (item) => {
      selectedBacklogForPO.value = item
      poModalMode.value = 'create'
      showPOModal.value = true
    }

    const viewPO = (item) => {
      selectedBacklogForPO.value = item
      poModalMode.value = 'view'
      showPOModal.value = true
    }

    const handlePOCreated = (poData) => {
      // Update the backlog item with the new PO ID
      const item = allBacklogItems.value.find(b => b.id === poData.backlog_item_id)
      if (item) {
        item.purchase_order_id = poData.id
        item.purchase_order = poData
      }
      showPOModal.value = false
    }

    // Watch for filter changes and reload data
    watch([selectedPeriod, selectedLocation, selectedCategory, selectedStatus], () => {
      loadData()
    })

    onMounted(loadData)

    return {
      t,
      loading,
      error,
      summary,
      ordersData,
      fillRate,
      statusData,
      orderHealthMetrics,
      categoryData,
      maxCategoryValue,
      orderTrendData,
      maxOrderCount,
      topProducts,
      backlogItems,
      calculatePercentage,
      getCircleSegment,
      getStockBadge,
      translateCategory,
      translateStockLevel,
      translatePriority,
      formatDate,
      revenueGoal,
      revenueGoalDisplay,
      showProductModal,
      selectedProduct,
      showProductDetail,
      showBacklogModal,
      selectedBacklogItem,
      showBacklogDetail,
      selectedPeriod,
      selectedCurrency: currentCurrency,
      formatCurrency,
      Math,
      translateProductName,
      translateWarehouse,
      showPOModal,
      selectedBacklogForPO,
      poModalMode,
      openPOModal,
      viewPO,
      handlePOCreated
    }
  }
}
</script>

<style scoped>
.dash-body {
  display: flex;
  flex-direction: column;
  gap: var(--space-6);
}

.page-header__title {
  margin: 0;
}

.section-title {
  margin: 0 0 var(--space-4);
  font-family: var(--font-display);
  font-weight: var(--fw-semibold);
  font-size: var(--text-md);
  color: var(--muted);
}

/* Loading / error placeholders */
.state-msg {
  padding: var(--space-12);
  text-align: center;
  color: var(--muted);
  font-size: var(--text-md);
}
.state-msg--error {
  color: var(--danger);
}

/* KPI progress under the value */
.stat-tile .progress {
  margin-top: var(--space-4);
}

/* ---- Order health card ---- */
.order-health {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--space-6);
  align-items: center;
}
.order-health__chart {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: var(--space-5);
}
.order-health__metrics {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--space-5);
}
.metric {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}
.metric__label {
  font-family: var(--font-mono);
  font-size: var(--text-2xs);
  font-weight: var(--fw-medium);
  letter-spacing: var(--tracking-wide);
  text-transform: uppercase;
  color: var(--faint);
}
.metric__value {
  font-family: var(--font-display);
  font-weight: var(--fw-bold);
  font-size: var(--text-lg);
  letter-spacing: var(--tracking-tight);
  color: var(--ink);
  font-variant-numeric: tabular-nums;
}
.metric__value--good { color: var(--success); }
.metric__value--warning { color: var(--warning); }
.metric__value--bad { color: var(--danger); }

/* Donut segment colors via tokens */
.donut__track { stroke: var(--surface-3); }
.donut__seg--delivered { stroke: var(--success); }
.donut__seg--shipped { stroke: var(--info); }
.donut__seg--processing { stroke: var(--warning); }
.donut__seg--backordered { stroke: var(--danger); }

.legend__swatch--delivered { background: var(--success); }
.legend__swatch--shipped { background: var(--info); }
.legend__swatch--processing { background: var(--warning); }
.legend__swatch--backordered { background: var(--danger); }

/* ---- Category bars ---- */
.bars {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
}
.bar-row__meta {
  display: flex;
  justify-content: space-between;
  gap: var(--space-3);
  margin-bottom: var(--space-2);
  font-size: var(--text-sm);
}
.bar-row__meta b {
  color: var(--ink-2);
  font-weight: var(--fw-medium);
}
.bar-row__meta span {
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  color: var(--muted);
}

/* ---- Tables ---- */
.table-scroll {
  overflow-x: auto;
}
.data-table td.cell-link {
  cursor: pointer;
}
.data-table tbody tr.row-link {
  cursor: pointer;
}
.text-danger {
  color: var(--danger);
  font-weight: var(--fw-semibold);
}
.text-warning {
  color: var(--warning);
  font-weight: var(--fw-semibold);
}

/* ---- Empty states ---- */
.no-data {
  padding: var(--space-8);
  text-align: center;
  color: var(--faint);
  font-size: var(--text-sm);
}
.no-shortage {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: var(--space-4);
  padding: var(--space-12);
  text-align: center;
}
.no-shortage__icon {
  color: var(--success);
}
.no-shortage__text {
  margin: 0;
  font-size: var(--text-md);
  font-weight: var(--fw-semibold);
  color: var(--success);
}
</style>
