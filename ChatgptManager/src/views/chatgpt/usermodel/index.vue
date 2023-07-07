<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="模型名称" prop="modelName">
        <el-input
          v-model="queryParams.modelName"
          placeholder="请输入模型名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="是否可用" prop="isUse">
        <el-select v-model="queryParams.isUse" placeholder="请选择是否可用" clearable>
          <el-option
            v-for="dict in dict.type.is_used"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="是否删除" prop="isDetele">
        <el-input
          v-model="queryParams.isDetele"
          placeholder="请输入是否删除"
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
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['chatgpt:usermodel:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['chatgpt:usermodel:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['chatgpt:usermodel:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['chatgpt:usermodel:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="usermodelList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="主键" align="center" prop="id" />
      <el-table-column label="模型名称" align="center" prop="modelName" />
      <el-table-column label="模型图片" align="center" prop="modelImage" width="100">
        <template slot-scope="scope">
          <image-preview :src="scope.row.modelImage" :width="50" :height="50"/>
        </template>
      </el-table-column>
      <el-table-column label="模型设定语" align="center" prop="modelContent" />
      <el-table-column label="是否可用" align="center" prop="isUse">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.is_used" :value="scope.row.isUse"/>
        </template>
      </el-table-column>
      <el-table-column label="是否删除" align="center" prop="isDetele" />
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['chatgpt:usermodel:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['chatgpt:usermodel:remove']"
          >删除</el-button>
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

    <!-- 添加或修改模型建设对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="模型名称" prop="modelName">
          <el-input v-model="form.modelName" placeholder="请输入模型名称" />
        </el-form-item>
        <el-form-item label="模型图片" prop="modelImage">
          <image-upload v-model="form.modelImage"/>
        </el-form-item>
        <el-form-item label="模型设定语" prop="modelContent">
          <el-input v-model="form.modelContent" type="textarea" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="是否可用" prop="isUse">
          <el-select v-model="form.isUse" placeholder="请选择是否可用">
            <el-option
              v-for="dict in dict.type.is_used"
              :key="dict.value"
              :label="dict.label"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="是否删除" prop="isDetele">
          <el-input v-model="form.isDetele" placeholder="请输入是否删除" />
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" type="textarea" placeholder="请输入内容" />
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
import { listUsermodel, getUsermodel, delUsermodel, addUsermodel, updateUsermodel } from "@/api/chatgpt/usermodel";

export default {
  name: "Usermodel",
  dicts: ['is_used'],
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
      // 模型建设表格数据
      usermodelList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        modelName: null,
        modelImage: null,
        modelContent: null,
        isUse: null,
        isDetele: null,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        modelName: [
          { required: true, message: "模型名称不能为空", trigger: "blur" }
        ],
        modelContent: [
          { required: true, message: "模型设定语不能为空", trigger: "blur" }
        ],
        isUse: [
          { required: true, message: "是否可用不能为空", trigger: "change" }
        ],
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询模型建设列表 */
    getList() {
      this.loading = true;
      listUsermodel(this.queryParams).then(response => {
        this.usermodelList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        modelName: null,
        modelImage: null,
        modelContent: null,
        isUse: null,
        isDetele: null,
        createBy: null,
        createTime: null,
        updateBy: null,
        updateTime: null,
        remark: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加模型建设";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getUsermodel(id).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改模型建设";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateUsermodel(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addUsermodel(this.form).then(response => {
              this.$modal.msgSuccess("新增成功");
              this.open = false;
              this.getList();
            });
          }
        }
      });
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids;
      this.$modal.confirm('是否确认删除模型建设编号为"' + ids + '"的数据项？').then(function() {
        return delUsermodel(ids);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('chatgpt/usermodel/export', {
        ...this.queryParams
      }, `usermodel_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
