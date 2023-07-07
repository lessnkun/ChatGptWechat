<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="卡号" prop="cardNumber">
        <el-input
          v-model="queryParams.cardNumber"
          placeholder="请输入卡号"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="是否使用" prop="isDetele">
        <el-select v-model="queryParams.isDetele" placeholder="请选择是否使用" clearable>
          <el-option
            v-for="dict in dict.type.is_used"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
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
          v-hasPermi="['chatgpt:rechargeablecardprocess:sys:export']"
        >导出</el-button>
        <el-button
          type="error"
          plain
          icon="el-icon-close"
          size="mini"
          @click="handleClose"
        >关闭</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="rechargeablecardprocessList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="主键" align="center" prop="id" />
      <el-table-column :show-overflow-tooltip="true" label="卡号" align="center" prop="cardNumber" />
      <el-table-column label="使用者ID" align="center" prop="userId" />
      <el-table-column label="批次ID" align="center" prop="batchId" />

      <el-table-column label="充值卡类型" align="center" prop="isDetele">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.recharge_card_type" :value="scope.row.cardType"/>
        </template>
      </el-table-column>
      <el-table-column label="是否使用" align="center" prop="isDetele">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.is_used" :value="scope.row.isDetele"/>
        </template>
      </el-table-column>
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['chatgpt:rechargeablecardprocess:remove']"
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

    <!-- 添加或修改充值卡详情对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="卡号" prop="cardNumber">
          <el-input v-model="form.cardNumber" placeholder="请输入卡号" />
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


    <!-- 修改 -->
    <el-dialog :title="title" :visible.sync="openEdit" width="500px" append-to-body>
      <el-form-item label="是否封卡" prop="isDetele">
        <el-select v-model="form.isDetele" placeholder="请选择是否封卡">
          <el-option
            v-for="dict in dict.type.is_used"
            :key="dict.value"
            :label="dict.label"
            :value="parseInt(dict.value)"
          ></el-option>
        </el-select>
      </el-form-item>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitFormEdit">确 定</el-button>
        <el-button @click="cancelEdit">取 消</el-button>
      </div>
    </el-dialog>


  </div>
</template>

<script>
import { listRechargeablecardprocesssys, getRechargeablecardprocesssys, delRechargeablecardprocesssys, addRechargeablecardprocesssys, updateRechargeablecardprocesssys } from "@/api/chatgpt/rechargeablecardprocess";

export default {
  name: "Rechargeablecardprocess",
  dicts: ['is_used','recharge_card_type'],
  data() {
    return {
      openEdit:false,
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
      // 充值卡详情表格数据
      rechargeablecardprocessList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      batchIdAdd:null,
      // 查询参数
      queryParams: {
        creatNumber:0,
        pageNum: 1,
        pageSize: 10,
        cardNumber: null,
        userId: null,
        batchId: null,
        isDetele: null,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      }
    };
  },
  created() {
    //获取参数
    this.queryParams.batchId = this.$route.query.batchId;
    this.getList();
  },
  methods: {
    /** 返回按钮操作 */
    handleClose() {
      const obj = { path: "/chatGpt/rechargeablecardmainsys" };
      this.$tab.closeOpenPage(obj);
    },
    /** 查询充值卡详情列表 */
    getList() {
      console.log(this.queryParams);
      this.loading = true;
      listRechargeablecardprocesssys(this.queryParams).then(response => {
        console.log(response);
        if (response.code=200){
          this.rechargeablecardprocessList = response.rows;
          this.total = response.total;
          this.loading = false;
        }else {
          this.$router.push({ path: "/chatGpt/rechargeablecardmainsys"});
        }

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
        cardNumber: null,
        userId: null,
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
      this.title = "生成批次内生成卡号只有单张";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getRechargeablecardprocesssys(id).then(response => {
        this.form = response.data;
        this.openEdit = true;
        this.title = "修改充值卡详情";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateRechargeablecardprocesssys(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addRechargeablecardprocesssys(this.form).then(response => {
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
      this.$modal.confirm('是否确认删除充值卡详情编号为"' + ids + '"的数据项？').then(function() {
        return delRechargeablecardprocesssys(ids);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('chatgpt/rechargeablecardprocess/sys/export', {
        ...this.queryParams
      }, `激活卡导出_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
