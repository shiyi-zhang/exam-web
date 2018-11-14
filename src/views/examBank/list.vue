<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.title" placeholder="试题内容" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-select v-model="listQuery.importance" placeholder="试题类型" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in importanceOptions" :key="item" :label="item" :value="item"/>
      </el-select>
      <el-select v-model="listQuery.importance" placeholder="专业" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in importanceOptions" :key="item" :label="item" :value="item"/>
      </el-select>
      <el-select v-model="listQuery.importance" placeholder="考点" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in importanceOptions" :key="item" :label="item" :value="item"/>
      </el-select>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">添加试题</el-button>
    </div>

    <el-table v-loading="listLoading" :key="tableKey" :data="list" border fit highlight-current-row style="width: 100%;">
      <el-table-column label="序号" align="center" width="65">
        <template slot-scope="scope">
          <span>{{ scope.row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="试题内容" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="试题类型" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.type }}</span>
        </template>
      </el-table-column>
      <el-table-column label="专业" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.role }}</span>
        </template>
      </el-table-column>
      <el-table-column label="考点" align="center">
        <template slot-scope="scope">
          <el-tag>{{ scope.row.id }}</el-tag>
          <el-tag>{{ scope.row.id }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="试卷名称" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.paper }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.timestamp }}</span>
        </template>
      </el-table-column>
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

    <el-dialog :visible.sync="dialogFormVisible" title="添加用户" width="70%">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="right" label-width="100px" style="width: 800px; margin-left:50px;">
        <el-form-item label="试题内容" prop="title">
          <el-input :autosize="{ minRows: 2, maxRows: 4}" v-model="temp.remark" type="textarea" placeholder="Please input" />
        </el-form-item>
        <el-form-item label="试卷名称" >
          <el-select placeholder="">
            <el-option :key="1" label="测试试卷"/>
          </el-select>
        </el-form-item>
        <el-form-item label="考点" >
          <el-select placeholder="">
            <el-option :key="1" label="测试考点"/>
          </el-select>
        </el-form-item>
        <el-form-item label="专业" prop="title">
          <el-select placeholder="">
            <el-option :key="1" label="测试专业"/>
          </el-select>
        </el-form-item>
        <el-form-item label="试题类型" prop="title">
          <el-select placeholder="">
            <el-option :key="1" label="单选"/>
            <el-option :key="2" label="多选"/>
          </el-select>
        </el-form-item>
        <el-form-item
          v-for="(domain, index) in domains"
          :label="'答案 ' + (index+1)"
          :key="domain.key"
          :prop="'domains.' + index + '.value'"
          :rules="{
            required: true, message: '答案不能为空', trigger: 'blur'
          }"
        >
          <el-input v-model="domain.value"/><el-button @click.prevent="removeDomain(domain)">删除</el-button>
        </el-form-item>
        <el-form-item label="解析" prop="title">
          <Tinymce ref="editor" :height="400" v-model="cont" />
        </el-form-item>
        <el-form-item label="备注">
          <el-input :autosize="{ minRows: 2, maxRows: 4}" v-model="temp.remark" type="textarea" placeholder="Please input" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addDomain">新增答案</el-button>
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">确定</el-button>
        <el-button v-else type="primary" @click="updateData">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import waves from '@/directive/waves' // 水波纹指令
import Tinymce from '@/components/Tinymce'
import request from '@/utils/request'

export default {
  name: 'ComplexTable',
  components: { Tinymce },
  directives: {
    waves
  },
  filters: {},
  data() {
    return {
      cent: '',
      tableKey: 0,
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10
      },
      temp: {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        type: '',
        status: 'published'
      },
      dialogFormVisible: false,
      dialogStatus: '',
      rules: {
        type: [
          { required: true, message: 'type is required', trigger: 'change' }
        ],
        timestamp: [
          {
            type: 'date',
            required: true,
            message: 'timestamp is required',
            trigger: 'change'
          }
        ],
        title: [
          { required: true, message: 'title is required', trigger: 'blur' }
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
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        status: 'published',
        type: ''
      }
    },
    paperDropdown() {
      request({
        url: '/list/exams',
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
    },
    removeDomain(item) {
      var index = this.domains.indexOf(item)
      if (index !== -1) {
        this.domains.splice(index, 1)
      }
    },
    addDomain() {
      this.domains.push({
        value: '',
        key: Date.now()
      })
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
          return
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
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
          tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
        }
      })
    },
    handleDelete(row) {
      this.$notify({
        title: '成功',
        message: '删除成功',
        type: 'success',
        duration: 2000
      })
      const index = this.list.indexOf(row)
      this.list.splice(index, 1)
    }
  }
}
</script>
