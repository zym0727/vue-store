<template>
  <div class="pageContainer">
    <el-card class="search-container">
      <slot name="table-search"></slot>
    </el-card>

    <div class="button-line">
      <slot name="table-action"></slot>
    </div>

    <el-table
        v-loading="loading"
        class="table-container"
        :data="tableData"
        max-height="530px"
        border
        stripe>
      <el-table-column
          align="center"
          v-for="column in columns"
          :key="column.prop"
          v-bind="column">
        <template v-if="$scopedSlots[column.prop]" #default="{ row }">
          <slot :name="column.prop" v-bind="row"></slot>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination">
      <el-pagination
          v-show="showPagination"
          background
          layout="prev, pager, next, sizes, jumper"
          @size-change="pageChange"
          @current-change="pageChange"
          :current-page.sync="currentPageNum"
          :page-size.sync="pageSizeNum"
          :total="total"
          :page-sizes="pageSizes">
      </el-pagination>
    </div>
  </div>
</template>

<script>
export default {
  name: "MyTablePage",
  props: {
    loading: {type: Boolean, default: false},
    tableData: {type: Array, default: () => []},
    columns: {type: Array, default: () => []},
    currentPage: {type: Number, default: 1},
    pageSize: {type: Number, default: 10},
    total: {type: Number, default: 0},
    pageSizes: {type: Array, default: () => [10, 20]},
    pageChange: {type: Function, default: () => {}},
    addRowFn: {type: Function, default: () => {}}
  },
  data() {
    return {
    }
  },
  computed: {
    currentPageNum: {
      get() {
        return this.currentPage
      },
      set(pageNum) {
        this.$emit('update:currentPage', pageNum)
      }
    },
    pageSizeNum: {
      get() {
        return this.pageSize
      },
      set(sizeNum) {
        this.$emit('update:pageSize', sizeNum)
      }
    },
    showPagination() {
      return this.tableData.length > 0;
    }
  },
}
</script>

<style scoped>
.pageContainer {
  width: 100%;
  height: 100%;
  overflow: auto;
}

.table-container {
  margin: 10px auto;
  width: 96%;
}

.pagination {
  display: flex;
  justify-content: center;
}

.search-container {
  margin: 20px auto 40px;
  width: 96%;
}

.button-line {
  display: flex;
  justify-content: space-between;
  margin: auto;
  width: 96%;
}
</style>