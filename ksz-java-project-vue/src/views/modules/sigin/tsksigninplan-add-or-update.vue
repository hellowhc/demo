<template>
  <div class="sigin-show">
    <el-dialog
      :title="!dataForm.id ? '创建签到方案' : '修改签到方案'"
      :close-on-click-modal="false"
      :visible.sync="visible"
    >
    <div class="form_sub">
<el-form
        :model="dataForm"
        :rules="dataRule"
        ref="dataForm"
        @keyup.enter.native="dataFormSubmit()"
      >
        <el-form-item label="方案名称" prop="signinName">
          <el-input v-model="dataForm.signinName"></el-input>
        </el-form-item>
        <el-form-item label="最小奖励" prop="startReward">
          <el-input v-model.number="dataForm.startReward"></el-input>
        </el-form-item>
        <el-form-item label="奖励增量" prop="incrementReward">
          <el-input v-model.number="dataForm.incrementReward"></el-input>
        </el-form-item>
        <el-form-item label="最大连续天数" prop="maxContinueDays">
          <el-input v-model.number="dataForm.maxContinueDays"></el-input>
        </el-form-item>
        <el-form-item label="方案开始时间" prop="signinStarttime">
          <el-date-picker
            v-model="dataForm.signinStarttime"
            align="right"
            type="date"
            placeholder="选择日期"
            value-format="yyyy-MM-dd HH:ss:mm"
          ></el-date-picker>
        </el-form-item>
        <el-form-item label="方案结束时间" prop="signinEndtime">
          <el-date-picker
            v-model="dataForm.signinEndtime"
            align="right"
            type="date"
            placeholder="选择日期"
            value-format="yyyy-MM-dd HH:ss:mm"
          ></el-date-picker>
        </el-form-item>
      </el-form>
    </div>
      
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit()" class="sub">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      pickerOptions1: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
      
      },
      dataForm: {
        id: 0,
        createTime: "",
        createUser: "",
        deleted: "",
        incrementReward: "",
        maxContinueDays: "",
        signinEndtime: "",
        signinStarttime: "",
        signinStatus: "",
        startReward: "",
        updateTime: "",
        updateUser: "",
        signinName: ""
      },
      dataRule: {
        incrementReward: [
          { required: true, message: "不能为空", trigger: "blur" },
          { type: "number", message: "为数字", trigger: "blur" }
        ],
        maxContinueDays: [
         { required: true, message: "不能为空", trigger: "blur" },
          { type: "number", message: "为数字", trigger: "blur" }
        ],
        signinEndtime: [
          {
            required: true,
            message: "不能为空",
            trigger: "blur"
          }
        ],
        signinStarttime: [
          {
            required: true,
            message: "不能为空",
            trigger: "blur"
          }
        ],
        startReward: [
          { required: true, message: "不能为空", trigger: "blur" },
          { type: "number", message: "为数字", trigger: "blur" }
        ],
        signinName: [{ required: true, message: "不能为空", trigger: "blur" }]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/sys/tsksigninplan/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.createTime = data.tskSigninPlan.createTime;
              this.dataForm.createUser = data.tskSigninPlan.createUser;
              this.dataForm.deleted = data.tskSigninPlan.deleted;
              this.dataForm.incrementReward =
                data.tskSigninPlan.incrementReward;
              this.dataForm.maxContinueDays =
                data.tskSigninPlan.maxContinueDays;
              this.dataForm.signinEndtime = data.tskSigninPlan.signinEndtime;
              this.dataForm.signinStarttime =
                data.tskSigninPlan.signinStarttime;
              this.dataForm.signinStatus = data.tskSigninPlan.signinStatus;
              this.dataForm.startReward = data.tskSigninPlan.startReward;
              this.dataForm.updateTime = data.tskSigninPlan.updateTime;
              this.dataForm.updateUser = data.tskSigninPlan.updateUser;
              this.dataForm.signinName = data.tskSigninPlan.signinName;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      //日期检测
      var d1 = new Date(this.dataForm.signinStarttime.replace(/\-/g, "\/"));  
      var d2 = new Date(this.dataForm.signinEndtime.replace(/\-/g, "\/"));  
      if(this.dataForm.signinStarttime!=""&&this.dataForm.signinEndtime!=""&&d1 >=d2)  
        {  
          this.$message.error(`开始时间${this.dataForm.signinStarttime}不能大于结束时间${this.dataForm.signinEndtime}`);
          return false;  
        }

      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/sys/tsksigninplan/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              signinStarttime: this.dataForm.signinEndtime,
              incrementReward: this.dataForm.incrementReward,
              maxContinueDays: this.dataForm.maxContinueDays,
              signinEndtime: this.dataForm.signinEndtime,
              startReward: this.dataForm.startReward,
              signinName: this.dataForm.signinName
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    }
  }
};
</script>
<style lang="less" scope>
.sigin-show {
  border:1px solid red;
  .form_sub{
    margin-top: 25px;
  }
  .sub {
    color: #fff;
    background-color: #1890ff;
    border-color: #1890ff;
    
  }
  .el-dialog__header {
    background: #fff;
    color: rgba(0, 0, 0, 0.65);
    border-bottom: 1px solid #e8e8e8;
    font-size: 16px;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
  }
  .el-dialog__footer {
    border-top: 1px solid #e8e8e8;
    border-bottom-left-radius: 10px;
    border-bottom-right-radius: 10px;
  }
  .el-picker-panel__sidebar {
    display: none;
    position: absolute;
    width: 100%;
    height: 30px;
    bottom: 0px;
    opacity: 0;
  }
  .el-dialog {
    width: 35% !important;
  }

  .el-form-item__label{
    color: rgba(0, 0, 0, 0.85) !important;
  }
  .el-dialog__body {
    padding: 5px 3px;
  }
  .el-form-item__label {
    display: inline-block;
    width: 120px !important;
    text-align: right;
  }
  .el-input {
    display: inline-block !important;
    width: 280px !important;
  }
  .el-input__inner {
    display: inline-block !important;
    width: 300px !important;
    flex: right;
  }
}
</style>