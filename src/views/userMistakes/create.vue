<template>
  <div class="createPost-container">

    <el-form ref="postForm" :model="postForm" :rules="rules" class="form-container">
      <sticky class-name="sub-navbar subtitle">
        <el-row>
          <el-col :span="20">
            <span>考点：第二章 课堂测试</span>
          </el-col>
          <el-col :span="4">
            <el-button v-loading="loading" type="danger">退出</el-button>
            <el-button v-loading="loading" style="margin-left: 10px;" type="success">交卷
            </el-button>
          </el-col>
        </el-row>
      </sticky>
      <div class="createPost-main-container">
        <el-row v-for="(item,index) in exam" :span="24" :key="index">
          <el-col>
            <el-card class="box-card" shadow="hover">
              <div slot="header" class="clearfix">
                <el-row>
                  <el-col :span="16">
                    <span v-if="item.type==='0'">题型：单选</span>
                    <span v-if="item.type==='1'">题型：多选</span>
                  </el-col>
                  <el-col :span="6">
                    <el-col :span="12">
                      <span>重要性：</span>
                    </el-col>
                    <el-col :span="12">
                      <el-rate v-model="item.importance" :max="5" disabled score-template="{value}" />
                    </el-col>
                  </el-col>
                  <el-col :span="2">
                    <el-button type="text" @click="setAnalysiShow(index)">查看解析</el-button>
                  </el-col>
                </el-row>
              </div>
              <el-row>
                <el-col :span="24">
                  <span>{{ index+1 +') '+item.content }}</span>
                </el-col>
                <el-col :span="24">
                  <br>
                  <el-form-item >
                    <el-radio-group v-if="item.type==='0'" v-model="answers[index]" >
                      <el-radio v-for="option in item.options" :label="option.id" :key="option.id">{{ option.label }}</el-radio>
                    </el-radio-group>
                    <el-checkbox-group v-if="item.type==='1'" v-model="answers[index]" >
                      <el-checkbox v-for="option in item.options" :label="option.id" :key="option.id">{{ option.label }}</el-checkbox>
                    </el-checkbox-group>
                  </el-form-item>
                </el-col>
                <el-col v-if="analysiShow[index]===true" :span="24">
                  <br>
                  <span style="color:#F56C6C;">解析：{{ item.analysis }}</span>
                </el-col>
              </el-row>
            </el-card>
          </el-col>
        </el-row>

      </div>
    </el-form>

  </div>
</template>

<script>
import Tinymce from '@/components/Tinymce'
import MDinput from '@/components/MDinput'
import Sticky from '@/components/Sticky' // 粘性header组件

const defaultForm = {
  status: 'draft',
  title: '', // 文章题目
  content: '', // 文章内容
  content_short: '', // 文章摘要
  display_time: undefined, // 前台展示时间
  id: undefined,
  platforms: ['a-platform'],
  comment_disabled: false
}

export default {
  name: 'ArticleDetail',
  components: { Tinymce, MDinput, Sticky },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value === '') {
        this.$message({
          message: rule.field + '为必传项',
          type: 'error'
        })
        callback(new Error(rule.field + '为必传项'))
      } else {
        callback()
      }
    }
    return {
      postForm: Object.assign({}, defaultForm),
      loading: false,
      userListOptions: [],
      answers: [],
      analysiShow: [false, false],
      exam: [
        {
          type: '1',
          content: '以科学的实验程序与方法研究人的心理现象和行为规律的学科是',
          options: [
            { label: 'a.普通心理学', id: 0 },
            { label: 'b.实验心理学', id: 1 },
            { label: 'c.认知心理学', id: 2 },
            { label: 'd.人格心理学', id: 3 }
          ],
          answers: ['1', '2'],
          importance: 2,
          analysis: '详细解析'
        },
        {
          type: '0',
          content: '以科学的实验程序与方法研究人的心理现象和行为规律的学科是',
          options: [
            { label: 'a.普通心理学', id: 0 },
            { label: 'b.实验心理学', id: 1 },
            { label: 'c.认知心理学', id: 2 },
            { label: 'd.人格心理学', id: 3 }
          ],
          answers: ['1'],
          importance: 2,
          analysis: '详细解析'
        }

      ],
      rules: {
        title: [{ validator: validateRequire }],
        content: [{ validator: validateRequire }]
      }
    }
  },
  computed: {
    contentShortLength() {
      return this.postForm.content_short.length
    }
  },
  created() {
    this.exam.forEach(e => {
      if (e.answers.length > 1) {
        this.answers.push([])
      } else {
        this.answers.push(undefined)
      }
    })
    console.log(this.answers)
  },
  methods: {
    draftForm() {
      this.$router.push({ path: '/testCentre/list' })
    },
    setAnalysiShow(index) {
      if (this.analysiShow[index]) {
        this.$set(this.analysiShow, index, false)
      } else {
        this.$set(this.analysiShow, index, true)
      }
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "src/styles/mixin.scss";

.demo_line_02 {
  height: 1px;
  border-bottom: 1px solid #ddd;
  text-align: center;
}

.createPost-container {
  position: relative;
  .createPost-main-container {
    padding: 40px 45px 20px 50px;
    .postInfo-container {
      position: relative;
      @include clearfix;
      margin-bottom: 10px;
      .postInfo-container-item {
        float: left;
      }
    }
    .editor-container {
      min-height: 500px;
      margin: 0 0 30px;
      .editor-upload-btn-container {
        text-align: right;
        margin-right: 10px;
        .editor-upload-btn {
          display: inline-block;
        }
      }
    }
  }
  .word-counter {
    width: 40px;
    position: absolute;
    right: -10px;
    top: 0px;
  }
}
</style>
