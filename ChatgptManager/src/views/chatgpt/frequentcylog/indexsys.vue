<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="操作名称" prop="operationName">
        <el-input
          v-model="queryParams.operationName"
          placeholder="请输入操作名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="日志类型" prop="operationType">
        <el-select v-model="queryParams.operationType" placeholder="请选择日志类型" clearable>
          <el-option
            v-for="dict in dict.type.operation_type"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="操作ID" prop="operationId">
        <el-input
          v-model="queryParams.operationId"
          placeholder="请输入操作ID"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="变化内容" prop="changeContent">
        <el-input
          v-model="queryParams.changeContent"
          placeholder="请输入变化内容"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="影响用户" prop="userId">
        <el-input
          v-model="queryParams.userId"
          placeholder="请输入影响用户"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['chatgpt:frequentcylog:sys:export']"
        >导出
        </el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="frequentcylogList" @selection-change="handleSelectionChange">
<!--      <el-table-column type="selection" width="55" align="center"/>-->
      <el-table-column label="主键" align="center" prop="id"/>
      <el-table-column label="操作名称" align="center" prop="operationName"/>
      <el-table-column label="日志类型" align="center" prop="operationType">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.operation_type" :value="scope.row.operationType"/>
        </template>
      </el-table-column>
      <el-table-column label="影响ID" align="center" prop="operationId"/>
      <el-table-column :show-overflow-tooltip="true" label="变化内容" align="center" prop="changeContent"/>
      <el-table-column label="影响用户" align="center" prop="userId"/>
<!--      <el-table-column label="是否删除" align="center" prop="isDetele"/>-->
      <el-table-column label="变化值(次/分钟)" align="center" prop="remark"/>
<!--      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">-->
<!--        <template slot-scope="scope">-->
<!--          <el-button-->
<!--            size="mini"-->
<!--            type="text"-->
<!--            icon="el-icon-edit"-->
<!--            @click="handleUpdate(scope.row)"-->
<!--            v-hasPermi="['chatgpt:frequentcylog:edit']"-->
<!--          >修改-->
<!--          </el-button>-->
<!--          <el-button-->
<!--            size="mini"-->
<!--            type="text"-->
<!--            icon="el-icon-delete"-->
<!--            @click="handleDelete(scope.row)"-->
<!--            v-hasPermi="['chatgpt:frequentcylog:remove']"-->
<!--          >删除-->
<!--          </el-button>-->
<!--        </template>-->
<!--      </el-table-column>-->
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改次数消耗日志对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="操作名称" prop="operationName">
          <el-input v-model="form.operationName" placeholder="请输入操作名称"/>
        </el-form-item>
        <el-form-item label="操作类型(1:对话)" prop="operationType">
          <el-select v-model="form.operationType" placeholder="请选择操作类型(1:对话)">
            <el-option
              v-for="dict in dict.type.operation_type"
              :key="dict.value"
              :label="dict.label"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="对应的ID,如果对话则可以对应到具体内容" prop="operationId">
          <el-input v-model="form.operationId" placeholder="请输入对应的ID,如果对话则可以对应到具体内容"/>
        </el-form-item>
        <el-form-item label="变化内容" prop="changeContent">
          <el-input v-model="form.changeContent" placeholder="请输入变化内容"/>
        </el-form-item>
        <el-form-item label="影响用户" prop="userId">
          <el-input v-model="form.userId" placeholder="请输入影响用户"/>
        </el-form-item>
        <el-form-item label="是否删除" prop="isDetele">
          <el-input v-model="form.isDetele" placeholder="请输入是否删除"/>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" type="textarea" placeholder="请输入内容"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listFrequentcylogsys,
  getFrequentcylogsys,
  delFrequentcylogsys,
  addFrequentcylogsys,
  updateFrequentcylogsys
} from '@/api/chatgpt/frequentcylog'

export default {
  name: 'Frequentcylog',
  dicts: ['operation_type'],
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
      // 次数消耗日志表格数据
      frequentcylogList: [],
      // 弹出层标题
      title: '',
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        operationName: null,
        operationType: null,
        operationId: null,
        changeContent: null,
        userId: null,
        isDetele: null
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {}
    }
  },
  created() {
    this.getList()
  },
  methods: {
    /** 查询次数消耗日志列表 */
    getList() {
      this.loading = true
      listFrequentcylogsys(this.queryParams).then(response => {
        this.frequentcylogList = response.rows
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
        operationName: null,
        operationType: null,
        operationId: null,
        changeContent: null,
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
      this.title = '添加次数消耗日志'
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset()
      const id = row.id || this.ids
      getFrequentcylogsys(id).then(response => {
        this.form = response.data
        this.open = true
        this.title = '修改次数消耗日志'
      })
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs['form'].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateFrequentcylogsys(this.form).then(response => {
              this.$modal.msgSuccess('修改成功')
              this.open = false
              this.getList()
            })
          } else {
            addFrequentcylogsys(this.form).then(response => {
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
      this.$modal.confirm('是否确认删除次数消耗日志编号为"' + ids + '"的数据项？').then(function() {
        return delFrequentcylogsys(ids)
      }).then(() => {
        this.getList()
        this.$modal.msgSuccess('删除成功')
      }).catch(() => {
      })
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('chatgpt/frequentcylog/sys/export', {
        ...this.queryParams
      }, `frequentcylog_${new Date().getTime()}.xlsx`)
    }
  }
}
</script>
