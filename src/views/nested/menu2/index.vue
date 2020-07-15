<template>
  <div>
    <MyTable 
      class="bms-tablepage__wrap-table"
      :operate-props='operateProps'
      :dataSource='dataSource'
      :columns='columns'
      :operates='operates'
      :total='total'
      :loading='loading'
      @handleChangePage='getList'
    >
      <!-- <template  v-slot:valText="slotProps">
        232323
        {{slotProps.data.row.name}}
      </template> -->
      <!-- <template  v-slot:valText>
        232323
      </template> -->
      <!-- <template  #valText>
        232323
      </template> -->
      <template  #valText="slotProps">
        232323{{slotProps.data.row.name}}
      </template>      
      <template  #beforeColum>
        <el-table-column
            prop="code12"
            label="测试"
          />
      </template>
    </MyTable>
  </div>
</template>
<script>
import MyTable from './components/MyTable'

export default {
  components:{MyTable},
  data () {
    return {
      // ...cloneDeep(this.$globalVar.baseTableData),//引入基础表格页定义数据searchForm,dataSource,pagination,total,
      operateProps:{
        fixed:'right',
        width:'220px'
      },
      columns:[
        {label: '头像',prop: 'headimg',image:true},
        {label: '姓名',prop: 'name'},
        {label: '用户编码',prop: 'code'},
        {
          label: '用户状态',
          prop: 'actived',
          render: (h, params) => {
            console.log(h)
            const type = params.row.actived == 1 ? '' : 'danger'
            const text = params.row.actived == 1 ? '激活' : '锁定'
            return <div><el-tag size='small' type={type}>{text}</el-tag><div>erer</div></div>
          }
        },
        {
          label: '值',
          slotName: 'valText'
        }     
      ],
      operates: [
        {
          label: '编辑',
          method: row => {
            this.handleEdit(row)
          }
        }
      ],
      dataSource: [
        {
          name: 'test1-test7.14',
          code: 'code1',
          code12: 'code2',
          actived: 0
        },
        {
          name: 'test2-test2',
          code: 'code3',
          code12: 'code2',
          actived: 1
        },
        {
          name: 'test3-test2',
          code: 'code3',
          code12: 'code2',
          actived: 1
        }
      ],
      total: 2,
      loading: false
    }
  },
  created(){
    // this.getList()
  },
  methods:{
    getList(isFresh){
      this.loading = true
      setTimeout(()=> {
        this.dataSource = []
        this.total = 0
        this.loading = false
      }, 500)
    },
    // 编辑
    handleEdit(row){
      console.log('编辑')
      console.log(row)
    }
  }
}
</script>