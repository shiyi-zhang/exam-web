<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.name" placeholder="专业名称" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">添加专业</el-button>
    </div>

    <el-table v-loading="listLoading" :data="list" border fit highlight-current-row style="width: 100%;">
      <el-table-column label="序号" align="center" width="65">
        <template slot-scope="scope">
          <span>{{ scope.row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="专业名称" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="备注" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.remark }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.timestamp }}</span>
        </template>
      </el-table-column>
      <!-- <el-table-column label="状态" width="90" align="center">
        <template slot-scope="scope">
          <el-tag :type="scope.row.status ">启用</el-tag>
        </template>
      </el-table-column> -->
      <el-table-column label="操作" align="center" width="140">
        <template slot-scope="scope">
          <el-button size="mini" type="text" @click="handleUpdate(scope.row)">编辑
          </el-button>
          <el-button size="mini" type="text" @click="handleDelete(scope.row)">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container">
      <el-pagination v-show="total>0" :current-page="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit" :total="total" background layout="total, sizes, prev, pager, next, jumper" @size-change="handleSizeChange" @current-change="handleCurrentChange" />
    </div>

    <el-dialog :visible.sync="dialogFormVisible" title="添加用户">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="100px" style="width: 400px; margin-left:50px;">
        <el-form-item label="专业名称" prop="name">
          <el-input v-model="temp.name" />
        </el-form-item>
        <el-form-item label="备注">
          <el-input :autosize="{ minRows: 2, maxRows: 4}" v-model="temp.remark" type="textarea" placeholder="Please input" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">确定</el-button>
        <el-button v-else type="primary" @click="updateData">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import waves from '@/directive/waves' // 水波纹指令
import request from '@/utils/request'

export default {
  name: 'ComplexTable',
  directives: {
    waves
  },
  filters: {},
  data() {
    return {
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        name: ''
      },
      temp: {},
      dialogFormVisible: false,
      dialogStatus: '',
      rules: {
        name: [
          { required: true, message: '专业必填', trigger: 'change' }
        ]
      }
    }
  },
  created() {
    this.resetTemp()
    this.getList()
  },
  methods: {
    resetTemp() {
      this.temp = {
        id: undefined,
        name: '',
        remark: ''
      }
    },
    getList() {
      this.listLoading = true
      request({
        url: '/special',
        method: 'get',
        params: {
          pageNo: this.listQuery.page,
          pageSize: this.listQuery.limit,
          name: this.listQuery.name
        }
      })
        .then(resData => {
          this.list = resData.data.list
          this.total = resData.data.total
        })
        .catch(err => {
          console.log(err)
        })
      setTimeout(() => {
        this.listLoading = false
      }, 0.5 * 1000)
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    handleSizeChange(val) {
      this.listQuery.limit = val
      this.getList()
    },
    handleCurrentChange(val) {
      this.listQuery.page = val
      this.getList()
    },

    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          request({
            url: '/special',
            method: 'post',
            data: tempData
          })
            .then(resData => {
              this.$notify({
                title: '成功',
                message: '添加成功',
                type: 'success',
                duration: 2000
              })
              this.getList()
              this.dialogFormVisible = false
            })
            .catch(err => {
              console.log(err)
            })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          request({
            url: '/special',
            method: 'put',
            data: tempData
          })
            .then(resData => {
              this.$notify({
                title: '成功',
                message: '更新成功',
                type: 'success',
                duration: 2000
              })
              this.getList()
              this.dialogFormVisible = false
            })
            .catch(err => {
              console.log(err)
            })
        }
      })
    },
    handleDelete(row) {
      request({
        url: '/special',
        method: 'delete',
        params: { id: row.id }
      })
        .then(resData => {
          this.$notify({
            title: '成功',
            message: '删除成功',
            type: 'success',
            duration: 2000
          })
          this.getList()
        })
        .catch(err => {
          console.log(err)
        })
    }
  }
}
</script>
