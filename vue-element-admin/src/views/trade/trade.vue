<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button class="filter-item" style="margin-left: 10px" type="primary">
        导出选中
      </el-button>

      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
        style="float: right"
      >
        搜索
      </el-button>
      <el-input
        v-model="listQuery.title"
        placeholder="用户名"
        style="width: 200px; float: right"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
    </div>

    <el-table
      :key="tableKey"
      ref="multipleTable"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55"> </el-table-column>
      <el-table-column label="订单号" align="center" width="100">
        <template slot-scope="{ row }">
          {{ row.no }}
        </template>
      </el-table-column>

      <el-table-column label="商品名称" align="left" width="400px">
        <template slot-scope="{ row }">
          {{ row.goods[0].title }}
        </template>
      </el-table-column>

      <el-table-column label="订单类型" width="100px" align="center">
        <template slot-scope="{ row }">
          {{ row.type == "default" ? "普通" : "拼团" }}
        </template>
      </el-table-column>
      <el-table-column label="状态" width="100px" align="center">
        <template slot-scope="{ row }">
          <el-button
            plain
            size="mini"
            :type="row.status == 'success' ? 'success' : 'danger'"
          >
            {{
              row.status == "success"
                ? "成功"
                : row.status == "pendding"
                ? "待支付"
                : "失败"
            }}
          </el-button>
        </template>
      </el-table-column>
      <el-table-column label="原付/实付(元)" width="150px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.total_price }}</span
          >/ <span style="color: red">{{ row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="支付方式" width="100px" align="center">
        <template slot-scope="{ row }">
          {{ row.pay_method }}
        </template>
      </el-table-column>

      <el-table-column label="创建/支付时间" width="200px" align="center">
        <template slot-scope="{ row }">
          <p>{{ row.created_time }}</p>
          <p>{{ row.pay_time }}</p>
        </template>
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        width="300"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-popover placement="top" width="160">
            <p>是否要删除该记录？</p>
            <div style="text-align: right; margin: 0">
              <el-button size="mini" type="text">取消</el-button>
              <!-- @click="handleDelete(row, $index)" -->
              <el-button
                type="primary"
                size="mini"
                @click="handleDelete(row, $index)"
                >确定</el-button
              >
            </div>
            <el-button slot="reference" type="danger">删除</el-button>
          </el-popover>
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
  </div>
</template>

<script>
import { fetchOrder, deleteOrder } from "@/api/pay";
import waves from "@/directive/waves"; // waves directive
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination

export default {
  name: "ComplexTable",
  components: { Pagination },
  directives: { waves },
  data() {
    return {
      // deleteVisible: false,
      tableKey: 0,
      list: [],
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        title: undefined,
      },
      multipleSelection: [],
    };
  },
  created() {
    this.getList();
  },
  methods: {
    handleDelete(row, index) {
      // console.log(this.temp);
      deleteOrder(row).then((response) => {
        this.$notify({
          title: "提示",
          message: "删除成功",
          type: "success",
          duration: 2000,
        });
        this.list.splice(index, 1);
      });
      // this.deleteVisible = false;
    },
    handleClose(done) {
      done();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    getList() {
      this.listLoading = true;
      fetchOrder(this.listQuery).then((res) => {
        console.log(res.data);
        if (res.code === 20000) {
          this.listLoading = false;
          this.list = res.data.items;
          this.total = res.data.total;
        }
      });
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
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
