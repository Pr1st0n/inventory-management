<template>
  <div class="filter-bar">
    <label class="fgroup">
      <span class="fgroup__key">{{ t('filters.timePeriod') }}</span>
      <select
        v-model="selectedPeriod"
        class="fselect"
        :class="{ 'is-armed': selectedPeriod !== 'all' }"
      >
        <option value="all">{{ t('filters.allMonths') }}</option>
        <option value="2025-01">{{ t('months.january') }}</option>
        <option value="2025-02">{{ t('months.february') }}</option>
        <option value="2025-03">{{ t('months.march') }}</option>
        <option value="2025-04">{{ t('months.april') }}</option>
        <option value="2025-05">{{ t('months.may') }}</option>
        <option value="2025-06">{{ t('months.june') }}</option>
        <option value="2025-07">{{ t('months.july') }}</option>
        <option value="2025-08">{{ t('months.august') }}</option>
        <option value="2025-09">{{ t('months.september') }}</option>
        <option value="2025-10">{{ t('months.october') }}</option>
        <option value="2025-11">{{ t('months.november') }}</option>
        <option value="2025-12">{{ t('months.december') }}</option>
      </select>
    </label>

    <label class="fgroup">
      <span class="fgroup__key">{{ t('filters.location') }}</span>
      <select
        v-model="selectedLocation"
        class="fselect"
        :class="{ 'is-armed': selectedLocation !== 'all' }"
      >
        <option value="all">{{ t('filters.all') }}</option>
        <option value="San Francisco">{{ t('warehouses.sanFrancisco') }}</option>
        <option value="London">{{ t('warehouses.london') }}</option>
        <option value="Tokyo">{{ t('warehouses.tokyo') }}</option>
      </select>
    </label>

    <label class="fgroup">
      <span class="fgroup__key">{{ t('filters.category') }}</span>
      <select
        v-model="selectedCategory"
        class="fselect"
        :class="{ 'is-armed': selectedCategory !== 'all' }"
      >
        <option value="all">{{ t('filters.all') }}</option>
        <option value="circuit boards">{{ t('categories.circuitBoards') }}</option>
        <option value="sensors">{{ t('categories.sensors') }}</option>
        <option value="actuators">{{ t('categories.actuators') }}</option>
        <option value="controllers">{{ t('categories.controllers') }}</option>
        <option value="power supplies">{{ t('categories.powerSupplies') }}</option>
      </select>
    </label>

    <label class="fgroup">
      <span class="fgroup__key">{{ t('filters.orderStatus') }}</span>
      <select
        v-model="selectedStatus"
        class="fselect"
        :class="{ 'is-armed': selectedStatus !== 'all' }"
      >
        <option value="all">{{ t('filters.all') }}</option>
        <option value="delivered">{{ t('status.delivered') }}</option>
        <option value="shipped">{{ t('status.shipped') }}</option>
        <option value="processing">{{ t('status.processing') }}</option>
        <option value="backordered">{{ t('status.backordered') }}</option>
      </select>
    </label>

    <button
      class="filter-bar__reset"
      @click="resetFilters"
      :disabled="!hasActiveFilters"
      title="Reset all filters"
    >
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a1 1 0 011.276.61A5.002 5.002 0 0014.001 13H11a1 1 0 110-2h5a1 1 0 011 1v5a1 1 0 11-2 0v-2.101a7.002 7.002 0 01-11.601-2.566 1 1 0 01.61-1.276z" clip-rule="evenodd" />
      </svg>
    </button>
  </div>
</template>

<script>
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'FilterBar',
  setup() {
    const {
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    } = useFilters()

    const { t } = useI18n()

    return {
      t,
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    }
  }
}
</script>

<style scoped>
.fgroup {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
}

.fgroup__key {
  font-family: var(--font-mono);
  font-size: var(--text-2xs);
  letter-spacing: var(--tracking-wide);
  text-transform: uppercase;
  color: var(--faint);
  white-space: nowrap;
}

.fselect {
  min-width: 140px;
}

.filter-bar__reset {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: var(--muted);
  text-decoration: none;
}

.filter-bar__reset svg {
  width: 16px;
  height: 16px;
}

.filter-bar__reset:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
</style>
