<template>
  <div class="app-container home">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>基本设置</span>
        <el-button
          style="float: right;margin-left: 10px"
          icon="el-icon-refresh"
          @click="refreshData()"
          v-hasPermi="['chatgpt:user:getConfigSys']"
          size="mini"
          type="primary"
        >刷新</el-button>

        <el-button
          type="success"
          v-if="!disable"
          style="float: right;"
          icon="el-icon-plus"
          @click="updateSettingFun()"
          v-hasPermi="['chatgpt:user:getConfigSys']"
          size="mini"
        >保存</el-button>


        <el-button
          type="warning"
          v-if="disable"
          style="float: right;margin-left: 10px"
          icon="el-icon-edit"
          @click="EditSettingFun()"
          v-hasPermi="['chatgpt:user:getConfigSys']"
          size="mini"
        >编辑</el-button>
        <el-button
          type="info"
          v-if="!disable"
          style="float: right;margin-left: 10px"
          icon="el-icon-close"
          @click="cancleSettingFun()"
          v-hasPermi="['chatgpt:user:getConfigSys']"
          size="mini"
        >取消</el-button>

        <el-button
          type="danger"
          plain
          icon="el-icon-refresh"
          style="float: right;margin-left: 10px"
          size="mini"
          @click="handleRefreshCache"
          v-hasPermi="['system:config:remove']"
        >刷新缓存</el-button>
        <el-button
          plain
          icon="el-icon-mobile"
          style="float: right;margin-left: 10px"
          size="mini"
          @click="dialogFormVisible = true"
          v-hasPermi="['system:config:remove']"
        >数据同步</el-button>
        <el-button
          plain
          icon="el-icon-mobile"
          style="float: right;margin-left: 10px"
          size="mini"
          @click="wordNotice = true"
          v-hasPermi="['system:config:remove']"
        >全局通告（实时）</el-button>
      </div>
      <el-form  v-loading="loading"  size="mini" label-width="100px"
               label-position="top"
      >
        <el-form-item v-for="(item,index) in formData " :label="index+1+':'+item.name">
          <el-input :disabled="disable" v-model="item.value" clearable :style="{width: '100%'}">
          </el-input>
          <div v-if="item.desc" style="margin-top: 10px">
            <el-alert
              type="success"
              :description="item.desc">
            </el-alert>
          </div>
        </el-form-item>
      </el-form>
    </el-card>




    <el-dialog title="同步数据" :visible.sync="dialogFormVisible">
      <el-form>
        <el-form-item label="同步方式" >
          <el-select v-model="updateType" placeholder="请选择同步方式">
            <el-option label="全量同步" value="1"></el-option>
            <el-option label="增量同步(推荐)" value="2"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="更新内容">
          <el-input type="textarea" rows="5" maxlength="-1" v-model="formUpdateDataStr" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="sysnConfig()">确 定</el-button>
      </div>
    </el-dialog>




    <el-dialog title="世界消息" :visible.sync="wordNotice">
      <el-form>
        <el-form-item label="消息内容">
          <el-input type="textarea" rows="5" maxlength="-1" v-model="wordText" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="wordNotice = false">取 消</el-button>
        <el-button type="primary" @click="wordNoticeIng()">发 送</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import {  updateSetting,refreshList,sysnConfigData,wordNoticeIng } from "@/api/chatgpt/setting";
import {  refreshCache } from "@/api/system/config";
export default {

  components: {},
  props: [],
  data() {
    return {
      formData: [],
      // 遮罩层
      loading: true,
      disable : true,
      dialogFormVisible:false,
      updateType:'2',
      formUpdateDataStr:'',
      wordNotice:false,
      wordText:''
    }
  },
  computed: {},
  watch: {},
  created() {
    this.refreshData()
  },
  mounted() {
  },
  methods: {
    wordNoticeIng(){
      this.loading = true
      wordNoticeIng(this.wordText).then(response => {
        if (response.code==200){
          this.$modal.msgSuccess("发送成功,请查看信息");
          this.loading = false
        }
      });
    },
    updateSettingFun(){
      this.loading = true
      updateSetting(this.formData).then(response => {
        if (response.code==200){
          this.$modal.msgSuccess("修改成功,已刷新缓存");
          this.loading = false
          this.disable = true
        }
      });
    },
    EditSettingFun(){
      this.disable = false
    },
    cancleSettingFun(){
      this.disable = true
      this.refreshData()
    },
    refreshData(){
      this.loading = true,
      refreshList().then(response => {
        if (response.code = 200){
          this.formData = response.data
          this.loading = false;
          this.disable = true
        }

      });
    },
  /** 刷新缓存按钮操作 */
  handleRefreshCache() {
    refreshCache().then(() => {
      this.$modal.msgSuccess("刷新成功");
    });
  },
    /** 刷新缓存按钮操作 */
    sysnConfig() {
      let req = false;
      let obj = null;
      try {
        obj =  JSON.parse(this.formUpdateDataStr);
        req= true;
      } catch (e) {
      }
    if (req){
      let param ={
        formData:obj,
        type:this.updateType
      }
      sysnConfigData(param).then(() => {
        this.dialogFormVisible = false
        this.refreshData();
        this.$modal.msgSuccess("同步完成");

      });
    }else {
      this.$modal.msgError("参数输入错误");
    }

    },

  }
}

</script>
<style>
.el-upload__tip {
  line-height: 1.2;
}

.home {
  font-family: "open sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 13px;
  color: #676a6c;
  overflow-x: hidden;
}
</style>
