<template>
  <section class="my-table--wrapper">
    <!-- table表格 -->
    <el-table
      :height="tableHeight"
      v-bind="tableProps"
      :border="border"
      :data="dataSource"
      v-loading="loading"
      v-on="tableEvents"
      ref="multipleTable"
      style="width: 100%;"
      :header-cell-style="{ fontWeight:'bold' }"
      :row-key="getRowKey"
      @selection-change="handleSelectionChange"
    >
      <!-- 配置数据前插槽 -->
      <slot name="beforeColum"></slot>
      <!-- 表格数据 -->
      <template v-for="(column, index) in columns">
        <!-- type属性列 -->
        <template v-if="column.type">
          <el-table-column
            :key="index"
            :type="column.type"
            v-bind="column.props"
            :label="column.label"
            :align="column.align"
            :width="column.width || '60px'"
            v-if="!column.isShow || (column.isShow && column.isShow())"
          />
        </template>
        <template v-else>
          <el-table-column
            :key="index"
            v-bind="column.props"
            :prop="column.prop"
            :label="column.label"
            :align="column.align"
            :width="column.width"
            :show-overflow-tooltip='column.tooltip || true'
            v-if="!column.isShow || (column.isShow && column.isShow())"
          >
            <template slot-scope="scope">
              <template v-if="!column.render">
                <!-- formatter -->
                <template v-if="column.formatter">
                  <span
                    v-html="column.formatter(scope.row, column, scope.$index)"
                    @click="column.click && column.click(scope.row, scope.$index)"
                  ></span>
                </template>
                <!-- 跳转 -->
                <template v-else-if="column.newjump">
                  <router-link
                    class="newjump"
                    v-bind="{ target : '_blank', ...column.target }"
                    :to="column.newjump(scope.row, column, scope.$index)"
                  >{{scope.row[column.prop] || column.content}}</router-link>
                </template>
                <!-- 图片 -->
                <template v-else-if="column.image">
                  <!-- <el-image style="width:80px; height: 80px;" fit='contain' :src="$globalVar.baseURL+scope.row[column.prop]">
                    <div slot="error" class="el-image__error">{{column.errMessage || '暂无图片'}}</div>
                  </el-image> -->
                </template>
                <!-- 操作按钮 -->
                <template v-else-if="column.operates">
                  <template v-for="(btn, key) in column.btn">
                    <span
                      :key="key"
                      v-if="!btn.isShow || (btn.isShow && btn.isShow(scope.row, scope.$index))"
                    >
                      <el-button
                        :size="btn.size || 'small'"
                        :type="btn.type || `text`"
                        :icon="btn.icon"
                        :plain="btn.plain"
                        :disabled="btn.disabled && btn.disabled(scope.row, scope.$index)"
                        @click.native.prevent="btn.method(scope.row, scope.$index)"
                      >{{ btn.label }}{{column.btn.length >= 2 ? '&nbsp;&nbsp;' : ''}}</el-button>
                    </span>
                  </template>
                </template>
                <slot v-else-if="column.slotName" :name="column.slotName" :data="scope"></slot>
                <template v-else>
                  <span
                    :style="column.click ? 'color: #409EFF; cursor: pointer;' : null"
                    @click="column.click && column.click(scope.row, scope.$index)"
                  >
                    {{ scope.row[column.prop] }}
                  </span>
                </template>
              </template>
              <!-- render -->
              <template v-else>
                <render :column="column" :row="scope.row" :render="column.render" :index="index"></render>
              </template>
            </template>
          </el-table-column>
        </template>
      </template>

      <!-- 配置数据后插槽 -->
      <slot name="afterColum"></slot>

      <!-- 操作按钮 -->
      <el-table-column
        label="操作"
        v-if="operates"
        v-bind="operateProps"
      >
        <template slot-scope="scope">
            <template v-for="(btn, key) in operates">
              <span
                class="table-btn"
                :key="key"
                v-if="!btn.isShow || (btn.isShow && btn.isShow(scope.row, scope.$index))"
              >
                <el-button
                  v-bind="btn.props && btn.props"
                  :class="[{'isDelete':btn.delete}]"
                  :size="btn.size"
                  :type="btn.type || `text`"
                  :icon="btn.icon"
                  :plain="btn.plain"
                  :disabled="btn.disabled && btn.disabled(scope.row, scope.$index)"
                  @click.native.prevent="btn.method(scope.row, scope.$index)"
                >
                  <span v-if="btn.formatter" v-html="btn.formatter(scope.row, scope.$index)"></span>
                  <template v-else>{{ btn.label }}</template>
                </el-button>
              </span>
            </template>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页部分 -->
    <!-- <div class="my-table--pagination">
      <el-pagination
        class="pagination"
        v-if="pagination"
        :total="total"
        :current-page="pagination.currentPage"
        :page-size="pagination.pageSize"
        @current-change="handleChangePage($event,'current')"
        @size-change="handleChangePage($event,'size')"
        layout="total, prev, pager, next, sizes, jumper"
      />
    </div> -->
  </section>
</template>

<script>
const methods = {
  // 复选框选中项
  handleSelectionChange (val) {
    this.multipleSelection = val
    this.$emit('handleSelectionChange', Array.from(val))
  },
  // 改变分页触发事件
  handleChangePage (val,type) {
    // console.log(val,type)
    if(type === 'size'){
      this.pagination.pageSize = val
      this.$emit('handleChangePage')
    }
    if(type === 'current'){
      this.pagination.currentPage = val
      this.$emit('handleChangePage')
    }
  },
  // 表格跨页多选
  getRowKey(row) {
    return row.id;
  },
  // 清空表格多选
  clearSelection(){
    this.$refs.multipleTable.clearSelection()
  }
}
export default {
  name: 'MyTable',
  props: {
    dataSource: {
      type: Array,
      default: () => []
    },
    columns: {
      type: Array,
      default: () => []
    },
    border: {
      type: Boolean,
      default: false
    },
    loading: {
      type: Boolean,
      default: false
    },
    slotcontent: {
      type: Boolean,
      default: false
    },
    operates: {
      type: Array
    },
    pagination: {
      type: Object,
      default: null
    },
    total: {
      type: Number,
      default: 0
    },
    tableProps: Object,
    operateProps:Object,
    tableEvents: Object
  },
  data () {
    return {
      multipleSelection: [],
    }
  },
  computed:{
    tableHeight(){
      if(!this.pagination){
        return 'calc(100%)'
      }else{
        return 'calc(100% - 56px)'
      }
    }
  },
  watch:{
    total(nV,oV){
      if(nV == (this.pagination.currentPage-1)*this.pagination.pageSize && nV != 0){
        this.pagination.currentPage -= 1;
        this.handleChangePage(this.pagination.currentPage, 'current')
      }
    }
  },
  methods,
  mounted () {
    this.$nextTick(() => {
      this.$emit('toggleRowSelection', this.$refs.multipleTable)
    })
  },
  components: {
    render: {
      functional: true,
      props: {
        row: Object,
        render: Function,
        index: Number,
        column: {
          type: Object,
          default: null
        }
      },
      render: (h, opt) => {
        console.log(h)
        const params = {
          row: opt.props.row,
          index: opt.props.index
        }
        if (opt.props.column) params.column = opt.props.column
        return opt.props.render(h, params)
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
// @import '~@/styles/bmsN/mixin.styl';
// >>>.el-table__body-wrapper{
//   scrollBar()
// }
// >>>.el-table__fixed-right-patch{
//   background $bms-background
// }
.my-table--wrapper {
  overflow: hidden;
}

.newjump {
  text-decoration: none;
  color: blue;
}
.isDelete{
  color red  
}
.my-table--pagination{
  box-sizing border-box
  padding-top 24px
  height 56px
  .pagination {
    text-align: center;
  }
}
.table-btn+.table-btn{
  margin-left 5px
}

</style>
