<template>
  <div class="card-container">
    <el-row :gutter="5" style="margin-bottom: 0px;">
      <!-- 左侧部门信息管理-->
      <el-col :span="6">
        <dept-mng @clickNode="clickNode"/>
      </el-col>
      <!-- 右侧员工信息列表 -->
      <el-col :span="18">
        <el-card :body-style="{ height: 'calc(100vh - 110px)'}">
          <el-button
            v-if="userType=='s'"
            type="primary"
            size="small"
            style="margin-bottom: 20px;"
            @click="showAddDialog = true"
          >新增员工</el-button>
          <div style="float:right">
            <el-input
              v-model="listQuery.empName"
              size="small"
              placeholder="请输入员工姓名"
              class="handle-select mr10"
              style="width: 300px;"
              @keyup.enter.native="handleFilter"
            />
            <el-button type="primary" icon="el-icon-search" size="small" @click="handleFilter">查询</el-button>
          </div>
          <el-table
            v-loading="listLoading"
            :data="tableData"
            border
            stripe
            highlight-current-row
            style="width: 100%"
            height="calc(100vh - 230px)"
          >
            <el-table-column prop="dept_code" label="部门编号" sortable show-overflow-tooltip/>
            <el-table-column prop="dept_name" label="部门名称" show-overflow-tooltip/>
            <el-table-column prop="emp_name" label="人员名称"/>
            <el-table-column prop="email" label="邮箱" show-overflow-tooltip/>
            <el-table-column prop="mobile" label="手机号" width="100px"/>
            <el-table-column label="操作" align="center" width="150px">
              <template slot-scope="scope">
                <el-button
                  v-if="userType=='s'"
                  size="mini"
                  type="primary"
                  plain
                  @click="editEmployee(scope.$index, scope.row)"
                >编辑</el-button>
                <el-button
                  v-if="userType=='s'"
                  size="mini"
                  type="danger"
                  @click="delEmployee(scope.$index, scope.row)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
          <pagination
            v-show="total>0"
            :total="total"
            :page.sync="listQuery.page"
            :limit.sync="listQuery.size"
            @pagination="query"
          />
        </el-card>
      </el-col>
    </el-row>
    <emp-add :show.sync="showAddDialog" :selected-dept-data="selectedDeptData" @saved="query"/>
    <emp-edit :show.sync="showEditDialog" :selected-dept-data="selectedDeptData" :row="currentRow" @saved="query" />
  </div>
</template>

<script>
import EmpAdd from '@/views/sys/employee/add.vue'
import EmpEdit from '@/views/sys/employee/edit.vue'
import deptMng from '@/views/sys/employee/deptMng.vue'
import Pagination from '@/components/Pagination'
import * as employeeApi from '@/api/employee.js'
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import { mapGetters } from 'vuex'
export default {
  components: { deptMng, Treeselect, EmpAdd, EmpEdit, Pagination },
  data() {
    return {
      showAddDialog: false,
      showEditDialog: false,
      selectedDeptData: {},
      currentRow: {},
      total: 0,
      listQuery: {
        empName: null,
        deptId: null,
        page: 1,
        size: 10
      },
      employeeFilter: {
        employeeName: ''
      },
      // 规格型号显示列表
      tableData: [],
      listLoading: false

    }
  },
  computed: {
    ...mapGetters(['userType'])
  },
  created() {
    this.query()
  },
  methods: {
    editEmployee(index, row) {
      this.showEditDialog = true
      this.currentRow = row
      if (this.listQuery.deptId == null) {
        this.selectedDeptData.deptId = row.dept_id
        this.selectedDeptData.deptNm = row.dept_name
      }
    },
    // 部门树操作
    clickNode(data) {
      this.selectedDeptData = {
        deptId: data.id,
        deptNm: data.label
      }
      this.listQuery.deptId = data.id
      this.getDeptEmpList(data.id)
    },
    query() {
      this.getDeptEmpList(this.listQuery.deptId)
    },
    getDeptEmpList(deptId) {
      this.listLoading = true
      this.listQuery.deptId = deptId
      employeeApi.getDeptEmpList(this.listQuery).then(rsp => {
        this.tableData = rsp.content
        this.total = rsp.totalElements
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getDeptEmpList(this.listQuery.deptId)
    },
    // 删除、修改状态
    delEmployee(index, row) {
      this.$confirm('确认删除该员工吗?', '提示', {
        type: 'warning'
      }).then(() => {
        employeeApi.delEmp(row.emp_id).then(res => {
          employeeApi.hasEmpList(this.listQuery.deptId).then(has => {
            if (has) {
              this.getDeptEmpList(this.listQuery.deptId)
            } else {
              this.getDeptEmpList()
            }
          })
          this.$message.success('删除成功')
        })
      })
    }
  }
}
</script>
