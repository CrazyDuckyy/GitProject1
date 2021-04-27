<template>
  <div class="container">
    <div class="header">
      <div class="balance">
        <div class="head">
          <p>可用余额(元)</p>
          <el-button type="primary" size="mini" waves>申请提现</el-button>
        </div>
        <hr />
        <div class="main">20.00</div>
      </div>
      <div class="income">
        <div class="head">
          <p>可用余额(元)</p>
          <!-- <el-button type="primary" size="mini">申请提现</el-button> -->
        </div>
        <hr />
        <div class="main">20.00</div>
      </div>
      <div class="toBeSettled">
        <div class="head">
          <p>可用余额(元)</p>
          <!-- <el-button type="primary" size="mini">申请提现</el-button> -->
        </div>
        <hr />
        <div class="main">20.00</div>
      </div>
      <div class="frozen">
        <div class="head">
          <p>可用余额(元)</p>
          <!-- <el-button type="primary" size="mini">申请提现</el-dbutton> -->
        </div>
        <hr />
        <div class="main">20.00</div>
      </div>
    </div>
    <div class="main1" style="margin-top: 50px; margin-left: 30px">
      <el-table
        :key="tableKey"
        v-loading="listLoading"
        :data="list"
        border
        fit
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column label="交易时间" width="180" align="center">
          <template slot-scope="{ row }">
            {{ row.created_time }}
          </template>
        </el-table-column>
        <el-table-column label="提款账号" width="180" align="center">
          <template slot-scope="{ row }">
            {{ row.account }}
          </template>
        </el-table-column>
        <el-table-column label="开户人" align="center">
          <template slot-scope="{ row }">
            {{ row.name }}
          </template></el-table-column
        >
        <el-table-column label="提现金额" align="center">
          <template slot-scope="{ row }">
            ￥{{ row.price }}
          </template></el-table-column
        >
        <el-table-column label="状态" align="center">
          <template slot-scope="{ row }">
            <el-button :type="row.status ? 'danger' : 'success'">
              {{ row.status ? "审核中" : "成功" }}
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
    </div>
  </div>
</template>

<script>
import { fetchAssets } from "@/api/pay";
import waves from "@/directive/waves"; // waves directive
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
export default {
  name: "assets",
  components: { Pagination },
  directives: { waves },
  data() {
    return {
      tableKey: 0,
      listLoading: false,
      list: [],
      total: 0,
      listQuery: {
        page: 1,
        limit: 10,
        title: undefined,
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      fetchAssets(this.listQuery).then((res) => {
        console.log(res.data);
        if (res.code === 20000) {
          this.listLoading = false;
          this.list = res.data.items;
          this.total = res.data.total;
        }
      });
    },
  },
};
</script>

<style  scoped>
.header {
  margin-left: 20px;
  margin-top: 10px;
  display: flex;
}
.balance,
.income,
.toBeSettled,
.frozen {
  width: 23%;
  margin: 0 1%;
  /* font-size: 30px; */
  border: 1px solid gray;
}

.head {
  font-size: 20px;
  height: 50px;
  display: flex;
  justify-content: space-around;
}
.main {
  font-size: 30px;
  height: 50px;
  padding-left: 30px;
}
</style>
