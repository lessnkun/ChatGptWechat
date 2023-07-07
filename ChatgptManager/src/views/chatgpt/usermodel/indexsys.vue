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
                          v-for="dict in dict.type.is_run"
                          :key="dict.value"
                          :label="dict.label"
                          :value="dict.value"
                      />
                    </el-select>
                  </el-form-item>



      <el-form-item label="模型分类" prop="dRoleId">
        <el-select v-model="queryParams.dRoleId" placeholder="请选择模型分类" clearable>
          <el-option
            v-for="dict in roleList"
            :key="dict.id"
            :label="dict.roleName"
            :value="dict.id"
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
            v-hasPermi="['chatgpt:usermodel:sys:add']"
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
            v-hasPermi="['chatgpt:usermodel:sys:edit']"
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
            v-hasPermi="['chatgpt:usermodel:sys:remove']"
        >删除</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="usermodelList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
              <el-table-column label="主键" align="center" prop="id" />
              <el-table-column label="模型名称" align="center" prop="modelName" />
<!--              <el-table-column label="模型图片" align="center" prop="modelImage" width="100">-->
<!--                <template slot-scope="scope">-->
<!--                  <image-preview :src="scope.row.modelImage" :width="50" :height="50"/>-->
<!--                </template>-->
<!--              </el-table-column>-->
              <el-table-column label="模型设定语" :show-overflow-tooltip="true"  align="center" prop="modelContent" />

      <el-table-column label="是否可用" align="center" prop="isUse">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.is_run" :value="scope.row.isUse"/>
        </template>
      </el-table-column>

      <el-table-column label="分类名称" align="center" prop="droleId">
        <template slot-scope="scope">
          <el-select disabled v-model="scope.row.droleId" placeholder="分类名称" clearable>
            <el-option
              v-for="dict in roleList"
              :key="dict.id"
              :label="dict.roleName"
              :value="dict.id"
            />
          </el-select>
        </template>
      </el-table-column>



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
    <el-dialog :title="title" :visible.sync="open" width="1000px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="120px">
                        <el-form-item label="模型名称" prop="modelName">
                          <el-input v-model="form.modelName" placeholder="请输入模型名称" />
                        </el-form-item>
                        <el-form-item label="模型设定语" prop="modelContent">
                          <el-input v-model="form.modelContent" type="textarea" placeholder="请输入内容" />
                        </el-form-item>
                        <el-form-item label="是否可用" prop="isUse">
                          <el-select v-model="form.isUse" placeholder="请选择是否可用">
                            <el-option
                                v-for="dict in dict.type.is_run"
                                :key="dict.value"
                                :label="dict.label"
                                :value="parseInt(dict.value)"
                            ></el-option>
                          </el-select>
                        </el-form-item>


        <el-form-item label="模型分类" prop="droleId">
          <el-select v-model="form.droleId" placeholder="请选择模型分类">
            <el-option
              v-for="dict in roleList"
              :key="dict.id"
              :label="dict.roleName"
              :value="parseInt(dict.id)"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="添加模型方式" prop="usermodeliNSERTType">
          <el-select v-model="usermodeliNSERTType" placeholder="请选择模型分类">
            <el-option
              v-for="dict in addModelType"
              :key="dict.value"
              :label="dict.name"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
        </el-form-item>


        <el-form-item v-if="usermodeliNSERTType==2" label="选择模型" prop="usermodeliNSERTType">

          <el-table
            height="400"
            :data="usermodelDataDefault"
            style="width: 100%">
            <el-table-column
              label="模型名称"
              width="180">
              <template slot-scope="scope">
                <div slot="reference" class="name-wrapper">
                  <el-tag size="medium">{{ scope.row.key }}</el-tag>
                </div>
              </template>
            </el-table-column>
            <el-table-column
              :show-overflow-tooltip="true"
              label="模型描述"
              width="500">
              <template slot-scope="scope">
                <div slot="reference" class="name-wrapper">
                  <el-tag size="medium">{{ scope.row.value }}</el-tag>
                </div>
              </template>
            </el-table-column>

            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  @click="selectModel( scope.row)">选择</el-button>
                </template>
            </el-table-column>
          </el-table>

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
  import {getModelAddress,listUsermodelsys, getUsermodelsys, delUsermodelsys, addUsermodelsys, updateUsermodelsys } from "@/api/chatgpt/usermodel";
  import { listRoleChangesys } from "@/api/chatgpt/roleChange";
  export default {
    name: "Usermodel",
        dicts: ['is_run'],
    data() {
      return {
        roleListDist:[],
        roleList:[],
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
          dRoleId: null,
        },
        queryParamRoles: {
          pageNum: 1,
          pageSize: 1000,
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
          droleId: [
            { required: true, message: "模型分类不能为空", trigger: "change" }
          ],


        },
        roleId:'',
        usermodelDataDefault:[],
        usermodeliNSERTType:1,
        addModelType:[
          {
          name:"自定义模型",
            value:1
        },{
            name:"官方提供模型",
            value:2
        }
        ]
      };
    },
    created() {
      this.getList();
      this.getRoleList();
    },
    methods: {
      selectModel(e){
        this.form.modelName = e.key;
        this.form.modelContent = e.value;
      },
      /** 查询模型建设列表 */
      getList() {
        this.loading = true;
        listUsermodelsys(this.queryParams).then(response => {
          this.usermodelList = response.rows;
          this.total = response.total;
          this.loading = false;
        });
      },
      getModelAddressData(){
        getModelAddress().then(response => {
          this.usermodelDataDefault = response.data;
        });
      },
      getRoleList() {
        listRoleChangesys(this.queryParamRoles).then(response => {
          this.roleList = response.rows;
          for (let i = 0;i< this.roleList.length;i++){
            console.log(this.roleList[i])
            let param = {
              "label": this.roleList[i].roleName,
               "value":this.roleList[i].id
            }
            this.roleListDist.push(param)
          }
      console.log(this.roleListDist)
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
        this.getModelAddressData()
        this.reset();
        const id = row.id || this.ids
        getUsermodelsys(id).then(response => {
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
              updateUsermodelsys(this.form).then(response => {
                this.$modal.msgSuccess("修改成功");
                this.open = false;
                this.getList();
              });
            } else {
              addUsermodelsys(this.form).then(response => {
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
          return delUsermodelsys(ids);
        }).then(() => {
          this.getList();
          this.$modal.msgSuccess("删除成功");
        }).catch(() => {});
      },
  /** 导出按钮操作 */
  handleExport() {
    this.download('chatgpt/usermodel/sys/export', {
      ...this.queryParams
    }, `usermodel_${new Date().getTime()}.xlsx`)
  }
  }
  };
</script>
