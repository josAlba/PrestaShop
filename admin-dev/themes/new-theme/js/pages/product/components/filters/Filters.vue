<!--**
 * Copyright since 2007 PrestaShop SA and Contributors
 * PrestaShop is an International Registered Trademark & Property of PrestaShop SA
 *
 * NOTICE OF LICENSE
 *
 * This source file is subject to the Open Software License (OSL 3.0)
 * that is bundled with this package in the file LICENSE.md.
 * It is also available through the world-wide-web at this URL:
 * https://opensource.org/licenses/OSL-3.0
 * If you did not receive a copy of the license and are unable to
 * obtain it through the world-wide-web, please send an email
 * to license@prestashop.com so we can send you a copy immediately.
 *
 * DISCLAIMER
 *
 * Do not edit or add to this file if you wish to upgrade PrestaShop to newer
 * versions in the future. If you wish to customize PrestaShop for your
 * needs please refer to https://devdocs.prestashop.com/ for more information.
 *
 * @author    PrestaShop SA and Contributors <contact@prestashop.com>
 * @copyright Since 2007 PrestaShop SA and Contributors
 * @license   https://opensource.org/licenses/OSL-3.0 Open Software License (OSL 3.0)
 *-->
<template>
  <div class="combinations-filters">
    <label
      for="caption-textarea"
      class="control-label"
    >Filter by:</label>

    <div class="combinations-filters-line">
      <filter-dropdown
        :key="filter.id"
        v-for="filter in filters"
        :children="filter.attributes"
        :parent-id="filter.id"
        :label="filter.name"
        @addFilter="addFilter"
        @removeFilter="removeFilter"
      />
      <button
        type="button"
        v-if="selectedFiltersNumber > 0"
        class="btn btn-outline-secondary combinations-filters-clear"
        @click="clearAll"
      >
        <i class="material-icons">close</i> Clear
        {{ selectedFiltersNumber }} filters
      </button>
    </div>
  </div>
</template>

<script>
  import {getProductAttributeGroups} from '@pages/product/services/attribute-groups';
  import FilterDropdown from '@pages/product/components/filters/FilterDropdown';
  import ProductEventMap from '@pages/product/product-event-map';

  const CombinationEvents = ProductEventMap.combinations;

  export default {
    name: 'Filters',
    data() {
      return {
        filters: [],
        selectedFilters: {},
      };
    },
    props: {
      productId: {
        type: Number,
        required: true,
      },
      eventEmitter: {
        type: Object,
        required: true,
      },
    },
    components: {
      FilterDropdown,
    },
    computed: {
      selectedFiltersNumber() {
        if (!this.selectedFilters) {
          return 0;
        }

        return Object.values(this.selectedFilters).reduce((total, attributes) => total + attributes.length, 0);
      },
    },
    mounted() {
      this.initFilters();
    },
    methods: {
      /**
       * This methods is used to initialize product filters
       */
      async initFilters() {
        try {
          this.filters = await getProductAttributeGroups(this.productId);
        } catch (error) {
          window.$.growl.error({message: error});
        }
      },
      addFilter(filter, parentId) {
        // If absent set new field with set method so that it's reactive
        if (!this.selectedFilters[parentId]) {
          this.$set(this.selectedFilters, parentId, []);
        }

        this.selectedFilters[parentId].push(filter);
        this.updateFilters();
      },
      removeFilter(filter, parentId) {
        if (!this.selectedFilters[parentId]) {
          return;
        }

        this.selectedFilters[parentId] = this.selectedFilters[parentId].filter(
          (e) => filter.id !== e.id,
        );
        this.updateFilters();
      },
      clearAll() {
        this.selectedFilters = [];
        this.$emit('clearAll');
        this.eventEmitter.emit(CombinationEvents.updateAttributeGroups, this.selectedFilters);
      },
      updateFilters() {
        this.eventEmitter.emit(CombinationEvents.updateAttributeGroups, this.selectedFilters);
      },
    },
  };
</script>

<style lang="scss" type="text/scss">
@import "~@scss/config/_settings.scss";

.combinations-filters {
  .control-label {
    font-weight: 600;
    color: #000;
    margin-botton: 1rem;
  }

  &-line {
    display: flex;
    align-items: flex-start;
    flex-wrap: wrap;
    margin: 0 -0.35rem;
  }
}
</style>
