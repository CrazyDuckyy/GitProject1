<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button
        class="filter-item"
        style="margin-left: 10px"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >
        增加收款账号
      </el-button>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column label="拼图案内容" align="left">
        <template slot-scope="{ row }">
          <div style="display: flex">
            <div>
              <img :src="row.value.cover" width="100px" height="50px" />
            </div>
            <div>
              <p style="margin: 0; line-height: 20px">{{ row.value.title }}</p>
              <p style="margin: 0; line-height: 20px">
                原始价格：<span style="color: red">￥{{ row.price }}</span>
              </p>
              <p style="margin: 0; line-height: 20px">
                拼团价格：<span style="color: red"
                  >￥{{ row.value.price }}</span
                >
              </p>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="成团人数" align="center" width="100">
        <template slot-scope="{ row }">
          <span>{{ row.p_num }}</span>
        </template>
      </el-table-column>
      <el-table-column label="拼团时限(小时)" align="center" width="100">
        <template slot-scope="{ row }">
          <span>{{ row.expire }}</span>
        </template>
      </el-table-column>
      <el-table-column label="拼团状态" align="center" width="100">
        <template slot-scope="{ row }">
          <!-- 0 是已下架 1是未开始？？  -->
          <span>{{ row.status }}</span>
        </template>
      </el-table-column>

      <el-table-column
        label="Actions"
        align="center"
        width="300"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button
            v-if="row.status != 'deleted'"
            size="mini"
            type="danger"
            @click="handleDelete(row, $index)"
          >
            下架
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />

    <!-- <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="100px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="账号" prop="account">
          <el-input v-model="temp.account" />
        </el-form-item>
        <el-form-item label="省市区">
          <el-input v-model="temp.province" />
        </el-form-item>
        <el-form-item label="详细地址" prop="path">
          <el-input v-model="temp.path" />
        </el-form-item>
        <el-form-item label="所属银行" prop="bank">
          <el-select v-model="temp.bank" placeholder="请选择">
            <el-option
              v-for="item in banks"
              :key="item.value"
              :label="item.text"
              :value="item.text"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input v-model="temp.name" />
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
        >
          确认
        </el-button>
      </div>
    </el-dialog> -->
  </div>
</template>

<script>
import { fetchGroup, createGroup, updateGroup } from "@/api/marketing";
import waves from "@/directive/waves"; // waves directive
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import Tinymce from "@/components/Tinymce";
// import { getBank } from "@/utils/bank";

export default {
  name: "ComplexTable",
  components: { Pagination, Tinymce },
  directives: { waves },
  data() {
    return {
      //   bank: [],
      tableKey: "",
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        title: undefined,
      },

      temp: {
        account: "",
        province: "",
        city: "",
        area: "",
        path: "",
        name: "",
      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "编辑",
        create: "创建",
      },
      dialogPvVisible: false,
      rules: {
        account: [
          {
            required: true,
            meseage: "账户不能为空",
            trigger: "blur",
          },
        ],
        bank: [
          {
            required: true,
            meseage: "所属银行不能为空",
            trigger: "change",
          },
        ],
        name: [
          {
            required: true,
            meseage: "开户人不能为空",
            trigger: "change",
          },
        ],
        path: [
          {
            required: true,
            meseage: "详细地址不能为空",
            trigger: "change",
          },
        ],
      },
      downloadLoading: false,
    };
  },
  created() {
    this.getList();
    // this.banks = getBank();
  },
  methods: {
    getList() {
      this.listLoading = true;
      fetchGroup(this.listQuery).then((res) => {
        console.log(res);
        if (res.code === 20000) {
          this.listLoading = false;
          this.list = res.data.items;
          this.total = res.data.total;
        }
      });
    },

    handleModifyStatus(row, status) {
      this.$message({
        message: "操作Success",
        type: "success",
      });
      row.status = !row.status;
      console.log(row.status);
    },

    resetTemp() {
      this.temp = {
        account: "",
        province: "",
        city: "",
        area: "",
        path: "",
        name: "",
      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        console.log(this.temp);
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
          this.temp.author = "vue-element-admin";
          createGroup(this.temp).then(() => {
            this.list.unshift(this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Created Successfully",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          tempData.timestamp = +new Date(tempData.timestamp); // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateGroup(tempData).then(() => {
            const index = this.list.findIndex((v) => v.id === this.temp.id);
            this.list.splice(index, 1, this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Update Successfully",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    handleDelete(row, index) {
      // console.log(this.temp);
      deletePayment(row).then((response) => {
        this.$notify({
          title: "提示",
          message: "删除成功",
          type: "success",
          duration: 2000,
        });
        this.list.splice(index, 1);
      });
    },
  },
};
</script>

<style  scoped>
.avatar-uploader .el-upload {
  border: 3px dashed #57525257;
  border-radius: 6px;

  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
