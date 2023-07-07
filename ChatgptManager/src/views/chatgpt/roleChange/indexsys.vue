<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
                  <el-form-item label="角色名称" prop="roleName">
                    <el-input
                        v-model="queryParams.roleName"
                        placeholder="请输入角色名称"
                        clearable
                        @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>

      <el-form-item label="是否系统角色" prop="userId">
        <el-select v-model="queryParams.userId" placeholder="请选择" clearable>
          <el-option
            v-for="dict in dict.type.role_is_self"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
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
            v-hasPermi="['chatgpt:roleChange:sys:add']"
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
            v-hasPermi="['chatgpt:roleChange:sys:edit']"
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
            v-hasPermi="['chatgpt:roleChange:sys:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="warning"
            plain
            icon="el-icon-download"
            size="mini"
            @click="handleExport"
            v-hasPermi="['chatgpt:roleChange:sys:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="roleChangeList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
              <el-table-column label="主键" align="center" prop="id" />
              <el-table-column label="角色名称" align="center" prop="roleName" />
<!--              <el-table-column label="角色图片" align="center" prop="roleImage" width="100">-->
<!--                <template slot-scope="scope">-->
<!--                  <image-preview :src="scope.row.roleImage" :width="50" :height="50"/>-->
<!--                </template>-->
<!--              </el-table-column>-->
              <el-table-column  :show-overflow-tooltip="true"  label="角色设定语" align="center" prop="roleContent" />
<!--              <el-table-column label="用户ID" align="center" prop="userId" />-->
              <el-table-column label="是否可用" align="center" prop="isUse">
                <template slot-scope="scope">
                      <dict-tag :options="dict.type.is_use" :value="scope.row.isUse"/>
                </template>
              </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
              size="mini"
              type="text"
              icon="el-icon-edit"
              @click="handleUpdate(scope.row)"
              v-hasPermi="['chatgpt:roleChange:edit']"
          >修改</el-button>
          <el-button
              size="mini"
              type="text"
              icon="el-icon-delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['chatgpt:roleChange:remove']"
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

    <!-- 添加或修改角色管理对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                        <el-form-item label="角色名称" prop="roleName">
                          <el-input v-model="form.roleName" placeholder="请输入角色名称" />
                        </el-form-item>
<!--                        <el-form-item label="角色图片" prop="roleImage">-->
<!--                          <image-upload max-count="1" v-model="form.roleImage"/>-->
<!--                        </el-form-item>-->
                        <el-form-item label="设置属性">
                          <el-input maxlength="-1" type="textarea" v-model="form.roleContent" placeholder="请输入属性(例如:你是一个医生,超级厉害的医生)" />
                        </el-form-item>
<!--                        <el-form-item label="用户ID" prop="userId">-->
<!--                          <el-input v-model="form.userId" placeholder="请输入用户ID" />-->
<!--                        </el-form-item>-->
<!--                        <el-form-item label="是否删除" prop="isDetele">-->
<!--                          <el-input v-model="form.isDetele" placeholder="请输入是否删除" />-->
<!--                        </el-form-item>-->
<!--                        <el-form-item label="备注" prop="remark">-->
<!--                          <el-input v-model="form.remark" type="textarea" placeholder="请输入内容" />-->
<!--                        </el-form-item>-->


                      <el-form-item label="是否使用" prop="isUse">
                        <el-select v-model="form.isUse" placeholder="请选择是否使用" clearable>
                          <el-option
                            v-for="dict in dict.type.is_use"
                            :key="dict.value"
                            :label="dict.label"
                            :value="dict.value"
                          />
                        </el-select>
                      </el-form-item>
<!--                        <el-form-item label="是否可用" prop="isUse">-->
<!--                          <el-input v-model="form.isUse" placeholder="请输入是否可用" />-->
<!--                        </el-form-item>-->
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import { listRoleChangesys, getRoleChangesys, delRoleChangesys, addRoleChangesys, updateRoleChangesys } from "@/api/chatgpt/roleChange";

  export default {
    name: "RoleChange",
    dicts: ['is_use','role_is_self'],
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
        // 角色管理表格数据
              roleChangeList: [],
        // 弹出层标题
        title: "",
        // 是否显示弹出层
        open: false,
        // 查询参数
        queryParams: {
          pageNum: 1,
          pageSize: 10,
                        roleName: null,
                        roleImage: null,
                        roleContent: null,
                        userId: null,
                        isDetele: null,
                        isUse: null,
        },
        // 表单参数
        form: {},
        // 表单校验
        rules: {
        }
      };
    },
    created() {
      this.getList();
    },
    methods: {
      /** 查询角色管理列表 */
      getList() {
        this.loading = true;
        listRoleChangesys(this.queryParams).then(response => {
          this.roleChangeList = response.rows;
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
                        roleName: null,
                        roleImage: null,
                        roleContent: null,
                        userId: null,
                        isDetele: null,
                        createBy: null,
                        createTime: null,
                        updateBy: null,
                        updateTime: null,
                        remark: null,
                        isUse: null
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
        this.title = "添加角色管理";
      },
      /** 修改按钮操作 */
      handleUpdate(row) {
        this.reset();
        const id = row.id || this.ids
        getRoleChangesys(id).then(response => {
          this.form = response.data;
          this.open = true;
          this.title = "修改角色管理";
        });
      },
      /** 提交按钮 */
      submitForm() {
        this.$refs["form"].validate(valid => {
          if (valid) {
            if (this.form.id != null) {
              updateRoleChangesys(this.form).then(response => {
                this.$modal.msgSuccess("修改成功");
                this.open = false;
                this.getList();
              });
            } else {
              addRoleChangesys(this.form).then(response => {
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
        this.$modal.confirm('是否确认删除角色管理编号为"' + ids + '"的数据项？').then(function() {
          return delRoleChangesys(ids);
        }).then(() => {
          this.getList();
          this.$modal.msgSuccess("删除成功");
        }).catch(() => {});
      },
  /** 导出按钮操作 */
  handleExport() {
    this.download('chatgpt/roleChange/sys/export', {
      ...this.queryParams
    }, `roleChange_${new Date().getTime()}.xlsx`)
  }
  }
  };
</script>
