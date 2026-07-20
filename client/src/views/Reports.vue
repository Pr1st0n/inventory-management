<template>
  <div class="reports">
    <header class="page-header">
      <div class="page-header__titles">
        <h1 class="page-header__title">Performance Reports</h1>
        <p class="page-header__subtitle">View quarterly performance metrics and monthly trends</p>
      </div>
    </header>

    <div v-if="loading" class="reports__loading">Loading reports...</div>
    <div v-else-if="error" class="reports__error">{{ error }}</div>
    <div v-else class="reports__body">
      <!-- Quarterly Performance -->
      <section class="card">
        <div class="card__head">
          <h3 class="card__title">Quarterly Performance</h3>
        </div>
        <table class="data-table">
          <thead>
            <tr>
              <th>Quarter</th>
              <th class="is-numeric">Total Orders</th>
              <th class="is-numeric">Total Revenue</th>
              <th class="is-numeric">Avg Order Value</th>
              <th class="is-numeric">Fulfillment Rate</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(q, index) in quarterlyData" :key="index">
              <td><span class="data-table__strong">{{ q.quarter }}</span></td>
              <td class="is-numeric"><span class="data-table__num">{{ q.total_orders }}</span></td>
              <td class="is-numeric"><span class="data-table__num">${{ formatNumber(q.total_revenue) }}</span></td>
              <td class="is-numeric"><span class="data-table__num">${{ formatNumber(q.avg_order_value) }}</span></td>
              <td class="is-numeric">
                <span :class="getFulfillmentClass(q.fulfillment_rate)">
                  {{ q.fulfillment_rate }}%
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </section>

      <!-- Monthly Trends Chart -->
      <section class="card">
        <div class="card__head">
          <h3 class="card__title">Monthly Revenue Trend</h3>
        </div>
        <div class="card__body">
          <div class="bar-chart">
            <div v-for="(month, index) in monthlyData" :key="index" class="bar-wrapper">
              <div class="bar-container">
                <div
                  class="bar"
                  :style="{ height: getBarHeight(month.revenue) + 'px' }"
                  :title="'$' + formatNumber(month.revenue)"
                ></div>
              </div>
              <div class="bar-label">{{ formatMonth(month.month) }}</div>
            </div>
          </div>
        </div>
      </section>

      <!-- Month-over-Month Comparison -->
      <section class="card">
        <div class="card__head">
          <h3 class="card__title">Month-over-Month Analysis</h3>
        </div>
        <table class="data-table">
          <thead>
            <tr>
              <th>Month</th>
              <th class="is-numeric">Orders</th>
              <th class="is-numeric">Revenue</th>
              <th class="is-numeric">Change</th>
              <th class="is-numeric">Growth Rate</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(month, index) in monthlyData" :key="index">
              <td><span class="data-table__strong">{{ formatMonth(month.month) }}</span></td>
              <td class="is-numeric"><span class="data-table__num">{{ month.order_count }}</span></td>
              <td class="is-numeric"><span class="data-table__num">${{ formatNumber(month.revenue) }}</span></td>
              <td class="is-numeric">
                <span v-if="index > 0" :class="getChangeClass(month.revenue, monthlyData[index - 1].revenue)">
                  {{ getChangeValue(month.revenue, monthlyData[index - 1].revenue) }}
                </span>
                <span v-else class="reports__dash">-</span>
              </td>
              <td class="is-numeric">
                <span v-if="index > 0" :class="getChangeClass(month.revenue, monthlyData[index - 1].revenue)">
                  {{ getGrowthRate(month.revenue, monthlyData[index - 1].revenue) }}
                </span>
                <span v-else class="reports__dash">-</span>
              </td>
            </tr>
          </tbody>
        </table>
      </section>

      <!-- Summary Stats -->
      <div class="grid grid--kpis">
        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">Total Revenue (YTD)</span>
          </div>
          <div class="stat-tile__value">${{ formatNumber(totalRevenue) }}</div>
        </div>
        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">Avg Monthly Revenue</span>
          </div>
          <div class="stat-tile__value">${{ formatNumber(avgMonthlyRevenue) }}</div>
        </div>
        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">Total Orders (YTD)</span>
          </div>
          <div class="stat-tile__value">{{ totalOrders }}</div>
        </div>
        <div class="stat-tile">
          <div class="stat-tile__head">
            <span class="stat-tile__label">Best Performing Quarter</span>
          </div>
          <div class="stat-tile__value">{{ bestQuarter }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Reports',
  data() {
    return {
      loading: true,
      error: null,
      quarterlyData: [],
      monthlyData: [],
      totalRevenue: 0,
      avgMonthlyRevenue: 0,
      totalOrders: 0,
      bestQuarter: ''
    }
  },
  mounted() {
    console.log('Reports component mounted')
    this.loadData()
  },
  methods: {
    async loadData() {
      console.log('Loading reports data...')
      try {
        this.loading = true

        // Fetch quarterly data
        console.log('Fetching quarterly data...')
        const quarterlyResponse = await axios.get('http://localhost:8001/api/reports/quarterly')
        this.quarterlyData = quarterlyResponse.data
        console.log('Quarterly data:', this.quarterlyData)

        // Fetch monthly data
        console.log('Fetching monthly data...')
        const monthlyResponse = await axios.get('http://localhost:8001/api/reports/monthly-trends')
        this.monthlyData = monthlyResponse.data
        console.log('Monthly data:', this.monthlyData)

        // Calculate summary stats
        console.log('Calculating summary stats...')
        this.calculateSummaryStats()
        console.log('Summary stats calculated')

      } catch (err) {
        console.log('Error loading reports:', err)
        this.error = 'Failed to load reports: ' + err.message
      } finally {
        this.loading = false
        console.log('Loading complete')
      }
    },

    calculateSummaryStats() {
      // Calculate total revenue
      var total = 0
      for (var i = 0; i < this.monthlyData.length; i++) {
        total = total + this.monthlyData[i].revenue
      }
      this.totalRevenue = total

      // Calculate average monthly revenue
      if (this.monthlyData.length > 0) {
        this.avgMonthlyRevenue = total / this.monthlyData.length
      } else {
        this.avgMonthlyRevenue = 0
      }

      // Calculate total orders
      var orders = 0
      for (var i = 0; i < this.monthlyData.length; i++) {
        orders = orders + this.monthlyData[i].order_count
      }
      this.totalOrders = orders

      // Find best quarter
      var bestQ = ''
      var bestRevenue = 0
      for (var i = 0; i < this.quarterlyData.length; i++) {
        if (this.quarterlyData[i].total_revenue > bestRevenue) {
          bestRevenue = this.quarterlyData[i].total_revenue
          bestQ = this.quarterlyData[i].quarter
        }
      }
      this.bestQuarter = bestQ
    },

    formatNumber(num) {
      console.log('Formatting number:', num)
      // Format number with commas
      var str = num.toString()
      var parts = str.split('.')
      var intPart = parts[0]
      var decPart = parts.length > 1 ? parts[1] : '00'

      var formatted = ''
      var count = 0
      for (var i = intPart.length - 1; i >= 0; i--) {
        if (count > 0 && count % 3 === 0) {
          formatted = ',' + formatted
        }
        formatted = intPart[i] + formatted
        count++
      }

      if (decPart.length === 1) {
        decPart = decPart + '0'
      }
      if (decPart.length > 2) {
        decPart = decPart.substring(0, 2)
      }

      return formatted + '.' + decPart
    },

    formatMonth(monthStr) {
      console.log('Formatting month:', monthStr)
      // Convert YYYY-MM to readable format
      var parts = monthStr.split('-')
      var year = parts[0]
      var month = parts[1]

      var monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
      var monthIndex = parseInt(month) - 1

      return monthNames[monthIndex] + ' ' + year
    },

    getBarHeight(revenue) {
      console.log('Calculating bar height for revenue:', revenue)
      // Calculate bar height (max height 200px)
      var maxRevenue = 0
      for (var i = 0; i < this.monthlyData.length; i++) {
        if (this.monthlyData[i].revenue > maxRevenue) {
          maxRevenue = this.monthlyData[i].revenue
        }
      }

      if (maxRevenue === 0) {
        return 0
      }

      var height = (revenue / maxRevenue) * 200
      return height
    },

    getFulfillmentClass(rate) {
      if (rate >= 90) {
        return 'badge success'
      } else if (rate >= 75) {
        return 'badge warning'
      } else {
        return 'badge danger'
      }
    },

    getChangeValue(current, previous) {
      var change = current - previous
      if (change > 0) {
        return '+$' + this.formatNumber(change)
      } else if (change < 0) {
        return '-$' + this.formatNumber(Math.abs(change))
      } else {
        return '$0.00'
      }
    },

    getChangeClass(current, previous) {
      var change = current - previous
      if (change > 0) {
        return 'positive-change'
      } else if (change < 0) {
        return 'negative-change'
      } else {
        return ''
      }
    },

    getGrowthRate(current, previous) {
      if (previous === 0) {
        return 'N/A'
      }

      var rate = ((current - previous) / previous) * 100
      var sign = rate > 0 ? '+' : ''

      return sign + rate.toFixed(1) + '%'
    }
  }
}
</script>

<style scoped>
.reports {
  padding: var(--space-0);
}

.page-header {
  position: static;
  background: transparent;
  border-bottom: 0;
  padding: 0;
}

.reports__body {
  display: flex;
  flex-direction: column;
  gap: var(--space-6);
}

.reports__loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--muted);
  font-size: var(--text-md);
}

.reports__error {
  background: var(--danger-soft);
  color: var(--danger);
  font-weight: var(--fw-medium);
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) var(--space-0);
}

.reports__dash {
  color: var(--faint);
}

/* Bar chart */
.bar-chart {
  display: flex;
  align-items: flex-end;
  justify-content: space-around;
  gap: var(--space-2);
  padding-bottom: var(--space-8);
}

.bar-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
  max-width: calc(var(--space-16) + var(--space-4));
}

.bar-container {
  height: calc(var(--space-16) * 3 + var(--space-8));
  display: flex;
  align-items: flex-end;
  width: 100%;
}

.bar {
  width: 100%;
  background: linear-gradient(to top, var(--accent), var(--accent-hover));
  border-radius: var(--radius-xs) var(--radius-xs) var(--space-0) var(--space-0);
  transition: background var(--transition-base);
  cursor: pointer;
}

.bar:hover {
  background: linear-gradient(to top, var(--accent-pressed), var(--accent));
}

.bar-label {
  margin-top: var(--space-6);
  font-size: var(--text-xs);
  color: var(--muted);
  text-align: center;
  transform: rotate(-45deg);
  white-space: nowrap;
}

/* Fulfillment / change indicators driven by methods returning legacy class names */
.badge.success { color: var(--success); background: var(--success-soft); }
.badge.warning { color: var(--warning); background: var(--warning-soft); }
.badge.danger { color: var(--danger); background: var(--danger-soft); }

.positive-change {
  color: var(--success);
  font-weight: var(--fw-semibold);
}

.negative-change {
  color: var(--danger);
  font-weight: var(--fw-semibold);
}
</style>
