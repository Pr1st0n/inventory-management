<template>
  <div class="demand rise">
    <div class="page-header__titles">
      <h1 class="page-header__title">{{ t('demand.title') }}</h1>
      <p class="page-header__subtitle">{{ t('demand.description') }}</p>
    </div>

    <div v-if="loading" class="state-message">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="state-message state-message--error">{{ error }}</div>
    <div v-else>
      <div class="trend-grid">
        <div class="card trend-card trend-card--up">
          <div class="card__body">
            <div class="trend-card__head">
              <span class="trend-card__icon trend-card__icon--up">↑</span>
              <div>
                <div class="trend-card__label">{{ t('demand.increasingDemand') }}</div>
                <div class="trend-card__count">{{ t('demand.itemsCount', { count: getForecastsByTrend('increasing').length }) }}</div>
              </div>
            </div>
            <ul class="trend-list">
              <li v-for="item in getForecastsByTrend('increasing').slice(0, 5)" :key="item.id" class="trend-list__item">
                <span class="trend-list__name">{{ item.item_name }}</span>
                <span class="delta delta--up">+{{ getChangePercent(item) }}%</span>
              </li>
              <li v-if="getForecastsByTrend('increasing').length > 5" class="trend-list__more">
                +{{ getForecastsByTrend('increasing').length - 5 }} {{ t('demand.more') }}
              </li>
            </ul>
          </div>
        </div>

        <div class="card trend-card trend-card--stable">
          <div class="card__body">
            <div class="trend-card__head">
              <span class="trend-card__icon trend-card__icon--stable">→</span>
              <div>
                <div class="trend-card__label">{{ t('demand.stableDemand') }}</div>
                <div class="trend-card__count">{{ t('demand.itemsCount', { count: getForecastsByTrend('stable').length }) }}</div>
              </div>
            </div>
            <ul class="trend-list">
              <li v-for="item in getForecastsByTrend('stable').slice(0, 5)" :key="item.id" class="trend-list__item">
                <span class="trend-list__name">{{ item.item_name }}</span>
                <span class="delta delta--neutral">{{ getChangePercent(item) }}%</span>
              </li>
              <li v-if="getForecastsByTrend('stable').length > 5" class="trend-list__more">
                +{{ getForecastsByTrend('stable').length - 5 }} {{ t('demand.more') }}
              </li>
            </ul>
          </div>
        </div>

        <div class="card trend-card trend-card--down">
          <div class="card__body">
            <div class="trend-card__head">
              <span class="trend-card__icon trend-card__icon--down">↓</span>
              <div>
                <div class="trend-card__label">{{ t('demand.decreasingDemand') }}</div>
                <div class="trend-card__count">{{ t('demand.itemsCount', { count: getForecastsByTrend('decreasing').length }) }}</div>
              </div>
            </div>
            <ul class="trend-list">
              <li v-for="item in getForecastsByTrend('decreasing').slice(0, 5)" :key="item.id" class="trend-list__item">
                <span class="trend-list__name">{{ item.item_name }}</span>
                <span class="delta delta--down">{{ getChangePercent(item) }}%</span>
              </li>
              <li v-if="getForecastsByTrend('decreasing').length > 5" class="trend-list__more">
                +{{ getForecastsByTrend('decreasing').length - 5 }} {{ t('demand.more') }}
              </li>
            </ul>
          </div>
        </div>
      </div>

      <div class="card">
        <div class="card__head">
          <h3 class="card__title">{{ t('demand.demandForecasts') }}</h3>
        </div>
        <table class="data-table">
          <thead>
            <tr>
              <th>{{ t('demand.table.sku') }}</th>
              <th>{{ t('demand.table.itemName') }}</th>
              <th class="is-numeric">{{ t('demand.table.currentDemand') }}</th>
              <th class="is-numeric">{{ t('demand.table.forecastedDemand') }}</th>
              <th class="is-numeric">{{ t('demand.table.change') }}</th>
              <th>{{ t('demand.table.trend') }}</th>
              <th>{{ t('demand.table.period') }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="forecast in forecasts" :key="forecast.id">
              <td><span class="data-table__code">{{ forecast.item_sku }}</span></td>
              <td>{{ forecast.item_name }}</td>
              <td class="is-numeric"><span class="data-table__num">{{ forecast.current_demand }}</span></td>
              <td class="is-numeric"><span class="data-table__num data-table__strong">{{ forecast.forecasted_demand }}</span></td>
              <td class="is-numeric">
                <span :class="['delta', getChangeClass(forecast)]">{{ getChangePercent(forecast) }}%</span>
              </td>
              <td>
                <span :class="['status', trendStatusClass(forecast.trend)]">
                  {{ t(`trends.${forecast.trend}`) }}
                </span>
              </td>
              <td>{{ translatePeriod(forecast.period) }}</td>
            </tr>
          </tbody>
        </table>
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
  name: 'Demand',
  setup() {
    const { t } = useI18n()
    const loading = ref(true)
    const error = ref(null)
    const allForecasts = ref([])
    const inventoryItems = ref([])

    // Use shared filters
    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    // Filter forecasts based on inventory filters
    const forecasts = computed(() => {
      if (selectedLocation.value === 'all' && selectedCategory.value === 'all') {
        return allForecasts.value
      }

      // Get SKUs of items that match the filters
      const validSkus = new Set(inventoryItems.value.map(item => item.sku))
      return allForecasts.value.filter(f => validSkus.has(f.item_sku))
    })

    const loadForecasts = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()

        const [forecastsData, inventoryData] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory({
            warehouse: filters.warehouse,
            category: filters.category
          })
        ])

        allForecasts.value = forecastsData
        inventoryItems.value = inventoryData
      } catch (err) {
        error.value = 'Failed to load demand forecasts: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch for filter changes and reload data
    watch([selectedLocation, selectedCategory], () => {
      loadForecasts()
    })

    const getForecastsByTrend = (trend) => {
      return forecasts.value.filter(f => f.trend === trend)
    }

    const getChangePercent = (forecast) => {
      const change = ((forecast.forecasted_demand - forecast.current_demand) / forecast.current_demand * 100).toFixed(1)
      return change > 0 ? `+${change}` : change
    }

    const getChangeClass = (forecast) => {
      const change = forecast.forecasted_demand - forecast.current_demand
      const changePercent = Math.abs((change / forecast.current_demand) * 100)

      // If change is within ±2%, consider it stable
      if (changePercent <= 2) {
        return 'delta--neutral'
      }

      if (change > 0) return 'delta--up'
      if (change < 0) return 'delta--down'
      return 'delta--neutral'
    }

    const trendStatusClass = (trend) => {
      if (trend === 'increasing') return 'status--success'
      if (trend === 'decreasing') return 'status--danger'
      return 'status--info'
    }

    const translatePeriod = (period) => {
      // Period values like "Next 3 months", "Q1 2025", "30 days", etc.
      const { currentLocale } = useI18n()
      if (currentLocale.value === 'ja') {
        return period
          .replace(/Next\s+/i, '次の')
          .replace(/\s+months/i, 'か月')
          .replace(/\s+month/i, 'か月')
          .replace(/\s+days/i, '日間')
          .replace(/\s+day/i, '日')
          .replace('Q1', '第1四半期')
          .replace('Q2', '第2四半期')
          .replace('Q3', '第3四半期')
          .replace('Q4', '第4四半期')
      }
      return period
    }

    onMounted(loadForecasts)

    return {
      t,
      loading,
      error,
      forecasts,
      getForecastsByTrend,
      getChangePercent,
      getChangeClass,
      trendStatusClass,
      translatePeriod
    }
  }
}
</script>

<style scoped>
.state-message {
  padding: var(--space-8);
  color: var(--muted);
  font-size: var(--text-sm);
  text-align: center;
}

.state-message--error {
  color: var(--danger);
  background: var(--danger-soft);
  border: 1px solid var(--danger-soft);
  border-radius: var(--radius-md);
}

.trend-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: var(--space-5);
  margin-bottom: var(--space-6);
}

@media (max-width: 900px) {
  .trend-grid {
    grid-template-columns: 1fr;
  }
}

.trend-card {
  border-left: var(--space-1) solid var(--border);
  transition: box-shadow var(--transition-base), transform var(--transition-base);
}

.trend-card:hover {
  box-shadow: var(--shadow-2);
  transform: translateY(-2px);
}

.trend-card--up { border-left-color: var(--success); }
.trend-card--stable { border-left-color: var(--info); }
.trend-card--down { border-left-color: var(--danger); }

.trend-card__head {
  display: flex;
  align-items: center;
  gap: var(--space-4);
  padding-bottom: var(--space-4);
  margin-bottom: var(--space-4);
  border-bottom: 1px solid var(--border);
}

.trend-card__icon {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  width: var(--space-10);
  height: var(--space-10);
  border-radius: var(--radius-md);
  font-family: var(--font-display);
  font-size: var(--text-xl);
  font-weight: var(--fw-bold);
}

.trend-card__icon--up { color: var(--success); background: var(--success-soft); }
.trend-card__icon--stable { color: var(--info); background: var(--info-soft); }
.trend-card__icon--down { color: var(--danger); background: var(--danger-soft); }

.trend-card__label {
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  font-weight: var(--fw-semibold);
  letter-spacing: var(--tracking-wide);
  text-transform: uppercase;
  color: var(--muted);
}

.trend-card__count {
  margin-top: var(--space-1);
  font-family: var(--font-display);
  font-size: var(--text-2xl);
  font-weight: var(--fw-bold);
  color: var(--ink);
}

.trend-list {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  margin: 0;
  padding: 0;
  list-style: none;
}

.trend-list__item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  background: var(--surface-2);
  border-radius: var(--radius-sm);
  transition: background var(--transition-fast);
}

.trend-list__item:hover { background: var(--surface-3); }

.trend-list__name {
  flex: 1;
  overflow: hidden;
  font-size: var(--text-sm);
  font-weight: var(--fw-medium);
  color: var(--ink-2);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.trend-list__more {
  padding: var(--space-2);
  font-size: var(--text-xs);
  font-style: italic;
  color: var(--faint);
  text-align: center;
}

.delta--neutral {
  color: var(--info);
  background: var(--info-soft);
}
</style>
