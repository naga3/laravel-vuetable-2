<template>
  <div>
    <vuetable
      ref="vuetable"
      api-url="api/user"
      :fields="['name', 'email', 'address']"
      pagination-path=""
      @vuetable:pagination-data="onPaginationData"
    ></vuetable>
    <vuetable-pagination
      ref="pagination"
      @vuetable-pagination:change-page="onChangePage"
      :css="css"
    ></vuetable-pagination>
  </div>
</template>

<script>
import {Vuetable, VuetablePagination} from 'vuetable-2'

export default {
  components: {
    Vuetable, VuetablePagination
  },
  data() {
    return {
      css: {
        wrapperClass: 'pagination',
        activeClass: 'active',
        disabledClass: 'disabled',
        icons: {
          first: 'glyphicon glyphicon-backward',
          prev: 'glyphicon glyphicon-triangle-left',
          next: 'glyphicon glyphicon-triangle-right',
          last: 'glyphicon glyphicon-forward'
        }
      }
    }
  },
  methods: {
    onPaginationData(data) {
      this.$refs.pagination.setPaginationData(data)
    },
    onChangePage(page) {
      this.$refs.vuetable.changePage(page)
    }
  }
}
</script>

<style>
.pagination a {
  cursor: pointer;
  border: 1px solid lightgray;
  padding: 5px 10px;
}
.pagination a.active {
  color: white;
  background-color: #337ab7;
}
.pagination a.btn-nav.disabled {
  color: lightgray;
  cursor: not-allowed;
}
</style>
