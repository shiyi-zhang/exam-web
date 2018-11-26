<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.content" placeholder="试题内容" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-select v-model="listQuery.type" placeholder="试题类型" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in examTypes" :key="item.id" :label="item.name" :value="item.id"/>
      </el-select>
      <el-select v-model="listQuery.special" placeholder="专业" clearable filterable style="width: 190px;" class="filter-item">
        <el-option v-for="item in specials" :key="item.id" :label="item.name" :value="item.id"/>
      </el-select>
      <el-select v-model="listQuery.paper" placeholder="试卷" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in papers" :key="item.id" :label="item.name" :value="item.id"/>
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
          <span>{{ scope.row.content }}</span>
        </template>
      </el-table-column>
      <el-table-column label="试题类型" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.type }}</span>
        </template>
      </el-table-column>
      <el-table-column label="专业" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.specialId }}</span>
        </template>
      </el-table-column>
      <el-table-column label="考点" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.testCentreId }}</span>
        </template>
      </el-table-column>
      <el-table-column label="试卷名称" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.paperId }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime }}</span>
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

    <el-dialog :visible.sync="dialogFormVisible" title="添加试题" width="70%">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="right" label-width="100px" style="width: 800px; margin-left:50px;">
        <el-form-item label="试题内容" >
          <el-input :autosize="{ minRows: 2, maxRows: 4}" v-model="temp.content" type="textarea" placeholder="Please input" />
        </el-form-item>
        <el-form-item label="试卷" >
          <el-select v-model="temp.paperId" placeholder="试卷" clearable class="filter-item">
            <el-option v-for="item in papers" :key="item.id" :label="item.name" :value="item.id"/>
          </el-select>
        </el-form-item>
        <el-form-item label="考点" >
          <el-select v-model="temp.testCentreId" placeholder="试卷" clearable class="filter-item">
            <el-option v-for="item in testCentres" :key="item.id" :label="item.name" :value="item.id"/>
          </el-select>
        </el-form-item>
        <el-form-item label="专业" >
          <el-select v-model="temp.specialId" placeholder="专业" clearable class="filter-item">
            <el-option v-for="item in specials" :key="item.id" :label="item.name" :value="item.id"/>
          </el-select>
        </el-form-item>
        <el-form-item label="试题类型" >
          <el-select v-model="temp.type" placeholder="试题类型" clearable class="filter-item">
            <el-option v-for="item in examTypes" :key="item.id" :label="item.name" :value="item.id"/>
          </el-select>
        </el-form-item>
        <el-form-item
          v-for="(answer, index) in temp.answers"
          :label="'答案 ' + (index+1)"
          :key="answer.index"
          :prop="'answers[' + index + ']'"
          :rules="{
            required: true, message: '答案不能为空', trigger: 'blur'
          }"
        >
          <el-checkbox v-model="answer.answer" label=""/>
          <el-input v-model="answer.label" style="width:420px;"/><el-button @click.prevent="removeDomain(answer)">删除</el-button>
        </el-form-item>
        <el-form-item label="解析" >
          <Tinymce ref="editor" :height="200" v-model="temp.analysis" />
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
import constant from '@/utils/constant'

export default {
  name: 'ComplexTable',
  components: { Tinymce },
  directives: {
    waves
  },
  filters: {},
  data() {
    return {
      examTypes: constant.examTypes,
      specials: [],
      papers: [],
      testCentres: [],
      exams: [],
      tableKey: 0,
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        content: '',
        type: '',
        special: '',
        paper: ''
      },
      temp: {},
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
  mounted() {

  },
  created() {
    this.resetTemp()
    this.getList()
    this.testCentresDropdown()
    this.specialsDropdown()
    this.paperDropdown()
  },
  methods: {
    resetTemp() {
      this.temp = {
        id: undefined,
        content: '',
        paperId: '',
        testCentreId: '',
        specialId: '',
        type: '',
        importance: '',
        remark: '',
        analysis: '',
        answers: [{ label: '', answer: false, level: 0 }]
      }
    },
    specialsDropdown() {
      request({
        url: '/list/specials',
        method: 'get'
      })
        .then(resData => {
          this.specials = resData.data
        })
        .catch(err => {
          console.log(err)
        })
    },
    testCentresDropdown() {
      request({
        url: '/list/testCentres',
        method: 'get'
      })
        .then(resData => {
          this.testCentres = resData.data
        })
        .catch(err => {
          console.log(err)
        })
    },
    paperDropdown() {
      request({
        url: '/list/papers',
        method: 'get'
      })
        .then(resData => {
          this.papers = resData.data
        })
        .catch(err => {
          console.log(err)
        })
    },
    removeDomain(item) {
      var index = this.temp.answers.indexOf(item)
      if (index !== -1) {
        this.temp.answers.splice(index, 1)
      }
    },
    addDomain() {
      var len = this.temp.answers.length
      this.temp.answers.push({
        label: '',
        answer: false,
        level: len
      })
    },
    getList() {
      this.listLoading = true
      request({
        url: '/exams',
        method: 'get',
        params: {
          pageNo: this.listQuery.page,
          pageSize: this.listQuery.limit,
          examContent: this.listQuery.content,
          type: this.listQuery.type,
          specialId: this.listQuery.special,
          paperId: this.listQuery.paper
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
            url: '/exams',
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
      this.resetTemp()
      request({
        url: '/exams/' + row.id,
        method: 'get'
      })
        .then(resData => {
          this.temp = resData.data
          this.dialogStatus = 'update'
          this.dialogFormVisible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].clearValidate()
          })
        })
        .catch(err => {
          console.log(err)
        })
    },
    updateData() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          request({
            url: '/exams',
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
        url: '/exams',
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
