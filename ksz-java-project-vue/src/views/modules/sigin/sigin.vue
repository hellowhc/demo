<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <div class="nav">
        <el-form-item>
            <span>按方案名称搜索:</span>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable @blur="getDataList()"></el-input>
      </el-form-item>
      </div>
      <el-button type="primary" @click="addOrUpdateHandle()" class="add">创建签到方案</el-button>
    </el-form>
    <el-table
      :data="dataList"
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column type="index" label="序号"></el-table-column>
      <el-table-column label="方案名称" prop="signinName"></el-table-column>
      <el-table-column label="最小奖励" prop="startReward"></el-table-column>
      <el-table-column label="奖励增量" prop="incrementReward"></el-table-column>
      <el-table-column label="最大连续天数" prop="maxContinueDays"></el-table-column>
      <el-table-column label="方案开始时间" prop="signinStarttime"></el-table-column>
      <el-table-column label="方案结束时间" prop="signinEndtime"></el-table-column>
      <el-table-column label="方案状态">
        <template slot-scope="scope">
          <el-switch
           v-model="scope.row.signinStatus"
            active-text="启"
            inactive-text="闭"
            active-color="#1890ff"
            inactive-color="#ff4949"
            :active-value="1"
            :nactive-value="0"
            @change="hande(scope.row)"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="创建者" prop="createUser"></el-table-column>
      <el-table-column label="创建时间" prop="createTime"></el-table-column>
      <el-table-column label="更新者" prop="updateUser"></el-table-column>
      <el-table-column label="更新时间" prop="updateTime"></el-table-column>
      <el-table-column label="操作" width="120px">
        <template slot-scope="scope" >
            <el-button-group>
              <div class="change">
              <el-button type="primary"  @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
              <el-button type="danger"  @click="deleteHandle(scope.row.id)">删除</el-button>
              </div>
            </el-button-group>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
  @size-change="sizeChangeHandle"
  @current-change="currentChangeHandle"
  :current-page="pageIndex"
  :page-sizes="[10, 20, 50, 100]"
  :page-size="pageSize"
  :total="totalPage"
  layout="total, sizes, prev, pager, next, jumper">
</el-pagination>

    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from "./tsksigninplan-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: ""
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    };
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    hande(row){
     let id = row.id;
     let status = row.signinStatus;
      if(!status){
        status = 0;
         this.$message.warning("方案状态已关闭");
      }else{
        this.$message.warning("方案状态已开启");
      }
       this.$http({
        url: this.$http.adornUrl(`/sys/tsksigninplan/updateSigninStatus/${id}/${status}`),
        method: "get",
        params: this.$http.adornParams({
          id:id,
          status: status,
        })
      }).then(({ data }) => {
        this.dataListLoading = false;
      });
      
    },
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/sys/tsksigninplan/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          signinName: this.dataForm.key
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          let arr = data.page.list;
          let len = arr.length;
          console.log(data.page.list)
          for(let i = 0; i<len; i ++){
            arr[i].createTime = arr[i].createTime.toString().split(" ")[0];
            arr[i].signinStarttime = arr[i].signinStarttime.toString().split(" ")[0];
            arr[i].signinEndtime = arr[i].signinEndtime.toString().split(" ")[0];
            if(arr[i].updateTime){
              arr[i].updateTime = arr[i].updateTime.toString().split(" ")[0];
            }
            
            console.log(1)
          }
          this.dataList = arr;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/sys/tsksigninplan/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              }
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    }
  }
};
</script>
<style lang="less" scope>
button.add {
  display: block;
  float: right;
  color: #fff;
  background-color: #1890ff;
  border-color: #1890ff;
  &:hover{
      background-color: #1890ff;
  border-color: #1890ff;
  }
}

.change {
  button {
    padding: 6px 0px;
    width: 47px;
    margin-left: 0px !important;
    letter-spacing: 2px;
    color: #fff;
    background-color: #1890ff;
    border-color: #1890ff;
    border-radius: 3px !important;
    &:nth-of-type(2) {
      color: #cf1322;
      background: #fafafa;
      border: 1px solid #fafafa;
    }
    &:hover{
      background-color: #1890ff;
      border-color: #1890ff;
    }
    &:nth-of-type(2):hover {
      background: #cf1322;
      color: #fff;
      border: 1px solid #cf1322;
    }
  }
}
.el-table {
  height: 100% !important;
  th {
    div {
      color: rgba(0, 0, 0, 0.85) !important;
      font-weight: 500 !important;
      text-align: left !important;
      font-weight: 500;
      font-size: 14px !important;
    }
  }
}
.el-switch__core{
  width: 50px;
}
.el-switch__label--left{
  position: absolute;
  left: 50%;
  display: none;
}
.el-switch__label--left.is-active{
  z-index: 999;
  color: white;
  display: inline-block;
}
.el-switch__label--right{
  position: absolute;
  right: 52%;
  display: none;
}
.el-switch__label--right.is-active{
  z-index: 999;
  color: white;
  display: inline-block;
}
.nav{
.el-form-item__content{
  width: 500px;
  .el-input--medium{
    width: 300px;
  }
 .el-input__inner{
    width: 200px;
    height: 30px;
  }
}
}

</style>