<template>
  <div class="app-container">


    <el-row :gutter="10" class="mb8">
<!--      <el-col :span="1.5">-->
<!--        <el-button-->
<!--          type="primary"-->
<!--          plain-->
<!--          icon="el-icon-plus"-->
<!--          size="mini"-->
<!--          @click="handleAdd"-->
<!--          v-hasPermi="['chatgpt:diglogueprocess:sys:add']"-->
<!--        >新增-->
<!--        </el-button>-->
<!--      </el-col>-->
<!--      <el-col :span="1.5">-->
<!--        <el-button-->
<!--          type="success"-->
<!--          plain-->
<!--          icon="el-icon-edit"-->
<!--          size="mini"-->
<!--          :disabled="single"-->
<!--          @click="handleUpdate"-->
<!--          v-hasPermi="['chatgpt:diglogueprocess:sys:edit']"-->
<!--        >修改-->
<!--        </el-button>-->
<!--      </el-col>-->
<!--      <el-col :span="1.5">-->
<!--        <el-button-->
<!--          type="danger"-->
<!--          plain-->
<!--          icon="el-icon-delete"-->
<!--          size="mini"-->
<!--          :disabled="multiple"-->
<!--          @click="handleDelete"-->
<!--          v-hasPermi="['chatgpt:diglogueprocess:sys:remove']"-->
<!--        >删除-->
<!--        </el-button>-->
<!--      </el-col>-->
<!--      <el-col :span="1.5">-->
<!--        <el-button-->
<!--          type="warning"-->
<!--          plain-->
<!--          icon="el-icon-download"-->
<!--          size="mini"-->
<!--          @click="handleExport"-->
<!--          v-hasPermi="['chatgpt:diglogueprocess:sys:export']"-->
<!--        >导出-->
<!--        </el-button>-->
<!--      </el-col>-->
      <el-button
        type="error"
        plain
        icon="el-icon-close"
        size="mini"
        @click="handleClose"
      >关闭</el-button>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="diglogueprocessList" @selection-change="handleSelectionChange">
<!--      <el-table-column type="selection" width="55" align="center"/>-->
      <el-table-column label="主键" align="center" prop="id"/>
      <el-table-column  :show-overflow-tooltip="true"  label="询问内容" align="center" prop="askContent"/>
      <el-table-column  :show-overflow-tooltip="true"  label="回答内容" align="center" prop="answerContent"/>
      <el-table-column label="用户ID" align="center" prop="userId"/>
      <el-table-column label="对话ID" align="center" prop="sessionId"/>
      <el-table-column label="是否删除" align="center" prop="isDetele"/>
      <el-table-column label="备注" align="center" prop="remark"/>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-view"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['chatgpt:diglogueprocess:edit']"
          >查看
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改充值卡-从对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="询问内容">
          <el-input type="textarea" :disabled="true" v-model="form.askContent" placeholder="请输入询问内容"/>
        </el-form-item>
        <el-form-item label="回答内容">
          <el-input  type="textarea" :disabled="true" v-model="form.answerContent" placeholder="请输入回答内容"/>
        </el-form-item>
<!--        <el-form-item label="用户ID" prop="userId">-->
<!--          <el-input  :disabled="true" v-model="form.userId" placeholder="请输入用户ID"/>-->
<!--        </el-form-item>-->
        <el-form-item label="对话ID" prop="sessionId">
          <el-input  :disabled="true" v-model="form.sessionId" placeholder="请输入对话ID"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">关 闭</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listDiglogueprocesssys,
  getDiglogueprocesssys,
  delDiglogueprocesssys,
  addDiglogueprocesssys,
  updateDiglogueprocesssys
} from '@/api/chatgpt/diglogueprocess'

export default {
  name: 'Diglogueprocess',
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 充值卡-从表格数据
      diglogueprocessList: [],
      // 弹出层标题
      title: '',
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        askContent: null,
        answerContent: null,
        userId: null,
        sessionId: null,
        isDetele: null
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {}
    }
  },
  created() {
    //获取参数
    this.queryParams.sessionId = this.$route.query.sessionId
    this.getList()
  },
  methods: {
    /** 查询充值卡-从列表 */
    getList() {
      this.loading = true
      listDiglogueprocesssys(this.queryParams).then(response => {
        this.diglogueprocessList = response.rows
        this.total = response.total
        this.loading = false
      })
    },
    // 取消按钮
    cancel() {
      this.open = false
      this.reset()
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        askContent: null,
        answerContent: null,
        userId: null,
        isDetele: null,
        createBy: null,
        createTime: null,
        updateBy: null,
        updateTime: null,
        remark: null
      }
      this.resetForm('form')
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1
      this.getList()
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm('queryForm')
      this.handleQuery()
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length !== 1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset()
      this.open = true
      this.title = '添加充值卡-从'
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset()
      const id = row.id || this.ids
      getDiglogueprocesssys(id).then(response => {
        this.form = response.data
        this.open = true
        this.title = '查看详情'
      })
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs['form'].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateDiglogueprocesssys(this.form).then(response => {
              this.$modal.msgSuccess('修改成功')
              this.open = false
              this.getList()
            })
          } else {
            addDiglogueprocesssys(this.form).then(response => {
              this.$modal.msgSuccess('新增成功')
              this.open = false
              this.getList()
            })
          }
        }
      })
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids
      this.$modal.confirm('是否确认删除充值卡-从编号为"' + ids + '"的数据项？').then(function() {
        return delDiglogueprocesssys(ids)
      }).then(() => {
        this.getList()
        this.$modal.msgSuccess('删除成功')
      }).catch(() => {
      })
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('chatgpt/diglogueprocess/sys/export', {
        ...this.queryParams
      }, `diglogueprocess_${new Date().getTime()}.xlsx`)
    },
    /** 返回按钮操作 */
    handleClose() {
      const obj = { path: "/chatGpt/digloguesys" };
      this.$tab.closeOpenPage(obj);
    },
  }
}
</script>
