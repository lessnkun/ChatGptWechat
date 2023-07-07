<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="批次ID" prop="batchId">
        <el-input
          v-model="queryParams.batchId"
          placeholder="请输入批次ID"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
<!--      <el-form-item label="用户ID" prop="userId">-->
<!--        <el-input-->
<!--          v-model="queryParams.userId"-->
<!--          placeholder="请输入用户ID"-->
<!--          clearable-->
<!--          @keyup.enter.native="handleQuery"-->
<!--        />-->
<!--      </el-form-item>-->
      <el-form-item label="是否删除" prop="isDetele">
        <el-select v-model="queryParams.isDetele" placeholder="请选择是否删除" clearable>
          <el-option
            v-for="dict in dict.type.is_use"
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
          v-hasPermi="['chatgpt:rechargeablecardprocess:add']"
        >生成激活卡</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['chatgpt:rechargeablecardmain:sys:remove']"
        >删除</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="rechargeablecardmainList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column v-if="false" hid label="主键" align="center" prop="id" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="200">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="批次ID" align="center" prop="batchId">
        <template slot-scope="scope">
          <span @click="goToDetilPage(scope.row.batchId)" style="cursor:pointer;color: #409eff">{{ scope.row.batchId }}</span>
        </template>
      </el-table-column>

<!--      <el-table-column label="用户ID" align="center" prop="userId" />-->
<!--      <el-table-column label="是否删除" align="center" prop="isDetele">-->
<!--        <template slot-scope="scope">-->
<!--          <dict-tag :options="dict.type.is_use" :value="scope.row.isDetele"/>-->
<!--        </template>-->
<!--      </el-table-column>-->
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDeleteBatchId(scope.row)"
            v-hasPermi="['chatgpt:rechargeablecardmain:remove']"
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

    <!-- 添加或修改充值卡-主对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="批次ID" prop="batchId">
          <el-input v-model="form.batchId" placeholder="请输入批次ID" />
        </el-form-item>
        <el-form-item label="用户ID" prop="userId">
          <el-input v-model="form.userId" placeholder="请输入用户ID" />
        </el-form-item>
        <el-form-item label="是否删除" prop="isDetele">
          <el-select v-model="form.isDetele" placeholder="请选择是否删除">
            <el-option
              v-for="dict in dict.type.is_use"
              :key="dict.value"
              :label="dict.label"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
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




    <!-- 添加或修改充值卡-主对话框 -->
    <el-dialog :title="title" :visible.sync="openProcess" width="500px" append-to-body>
      <el-form ref="formProcess" :model="formProcess" :rules="rulesProcess" label-width="80px">
        <el-form-item  label="充值类型" prop="cardType">
          <el-select v-model="formProcess.cardType" placeholder="请选择充值卡类型">
            <el-option
              v-for="dict in dict.type.recharge_card_type"
              :key="dict.value"
              :label="dict.label"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item v-if="formProcess.cardType==1" label="次数" prop="addNum">
          <el-input type="number" v-model="formProcess.addNum" placeholder="请输入批次ID" />
        </el-form-item>
        <el-form-item  v-if="formProcess.cardType==2" label="时间(分钟)" prop="addDate">
          <el-input type="number"  v-model="formProcess.addDate" placeholder="请输入用户ID" />
        </el-form-item>
        <el-form-item   label="生成数量" prop="creatNumber">
          <el-input type="number"  v-model="formProcess.creatNumber" placeholder="请输入用户ID" />
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="formProcess.remark" type="textarea" placeholder="请输入内容" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitFormProcess">确 定</el-button>
        <el-button @click="cancelProcess">取 消</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import { deleteTbRechargeableCardMainByBathIdsys,listRechargeablecardmainsys, getRechargeablecardmainsys, delRechargeablecardmainsys, addRechargeablecardmainsys, updateRechargeablecardmainsys } from "@/api/chatgpt/rechargeablecardmain";
import { addRechargeablecardprocesssys } from "@/api/chatgpt/rechargeablecardprocess";

export default {
  name: "Rechargeablecardmain",
  dicts: ['is_use','recharge_card_type'],
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
      // 充值卡-主表格数据
      rechargeablecardmainList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      openProcess: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        batchId: null,
        userId: null,
        isDetele: null,
      },
      // 表单参数
      form: {},
      formProcess: {},
      rulesProcess: {},
      // 表单校验
      rules: {
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    goToDetilPage(e){
      this.$router.push({ path: "/chatGpt/rechargeablecardprocesssys", query: {batchId: e} });
    },
    /** 查询充值卡-主列表 */
    getList() {
      this.loading = true;
      listRechargeablecardmainsys(this.queryParams).then(response => {
        this.rechargeablecardmainList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },

    cancelProcess() {
      this.openProcess = false;
      this.resetProcess();
    },

    // 表单重置
    reset() {
      this.form = {
        id: null,
        batchId: null,
        userId: null,
        isDetele: null,
        createBy: null,
        createTime: null,
        updateBy: null,
        updateTime: null,
        remark: null,
        cardType:null,
        addNum:null,
        addDate:null,
      };
      this.resetForm("form");
    },

    // 重置详情页表单
    resetProcess() {
      this.formProcess = {
        cardType: null,
        addNum: null,
        addDate: null,
        remark:null,
        creatNumber:null
      };
      this.resetForm("formProcess");
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
      this.resetProcess();
      this.openProcess = true;
      this.title = "添加充值卡";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getRechargeablecardmainsys(id).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改充值卡-主";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateRechargeablecardmainsys(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addRechargeablecardmainsys(this.form).then(response => {
              this.$modal.msgSuccess("新增成功");
              this.open = false;
              this.getList();
            });
          }
        }
      });
    },

    /** 提交按钮 */
    submitFormProcess() {
      this.$refs["formProcess"].validate(valid => {
        if (valid) {
          addRechargeablecardprocesssys(this.formProcess).then(response => {
            this.$modal.msgSuccess("新增成功");
            this.openProcess = false;
            this.getList();
          });
        }
      });
    },

    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids;
      this.$modal.confirm('是否确认删除充值卡-主编号为"' + ids + '"的数据项？').then(function() {
        return delRechargeablecardmainsys(ids);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },


    handleDeleteBatchId(row) {
      const batchId = row.batchId;
      this.$modal.confirm('是否确认删除充值卡批号为"' + batchId + '"的数据项？').then(function() {
        return deleteTbRechargeableCardMainByBathIdsys(batchId);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('chatgpt/rechargeablecardmain/sys/export', {
        ...this.queryParams
      }, `rechargeablecardmain_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
