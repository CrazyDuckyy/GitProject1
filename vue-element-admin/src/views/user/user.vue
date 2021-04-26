<template>
  <div class="app-container">
    <div class="filter-container">
      <el-dropdown>
        <el-button type="danger">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="changeCommentStatus(true)"
            >禁止访问</el-dropdown-item
          >
          <el-dropdown-item @click.native="changeAssessStatus(true)"
            >禁止评论</el-dropdown-item
          >
        </el-dropdown-menu>
      </el-dropdown>
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
      <el-table-column label="用户" align="center">
        <template slot-scope="{ row }">
          <div>
            <img
              :src="row.user.avatar"
              style="width: 50px; height: 50px; float: left; border-radius: 50%"
            />
          </div>
          <span style="float: left; padding-top: 10px; padding-left: 5px">{{
            row.user.username
          }}</span>
        </template>
      </el-table-column>
      <el-table-column label="消费总额" align="center" width="100px">
        <template slot-scope="{ row }">
          {{ row.total_consume }}
        </template>
      </el-table-column>

      <el-table-column label="创建时间" width="150px" align="center">
        <template slot-scope="{ row }">
          {{ row.created_time }}
        </template>
      </el-table-column>
      <el-table-column
        label="Actions"
        align="center"
        width="300"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="openInfo(row.user.id)">
            详情
          </el-button>

          <el-button type="success" size="mini"> 联系用户 </el-button>

          <el-dropdown style="padding-left: 10px">
            <el-button type="danger" size="mini">
              黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="changeCommentStatus(true)"
                >禁止访问</el-dropdown-item
              >
              <el-dropdown-item @click.native="changeAssessStatus(true)"
                >禁止评论</el-dropdown-item
              >
            </el-dropdown-menu>
          </el-dropdown>
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

    <info ref="info"></info>
  </div>
</template>

<script>
import { fetchList, changeCommentStatus, changeAccessStatus } from "@/api/user";
import waves from "@/directive/waves"; // waves directive
import { parseTime } from "@/utils";
import info from "./info";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination

export default {
  name: "ComplexTable",
  components: { Pagination, info },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: "success",
        draft: "info",
        deleted: "danger",
      };
      return statusMap[status];
    },
  },
  data() {
    return {
      activeName: "frist",
      dialogVisible: false,
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
    openInfo(id) {
      this.$refs.info.open(id);
      // console.log(this.$refs.info);
    },
    handleClose(done) {
      done();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    getList() {
      this.listLoading = true;
      fetchList(this.listQuery).then((res) => {
        console.log(res.data);
        if (res.code === 20000) {
          (this.listLoading = false),
            (this.list = res.data.items),
            (this.total = res.data.total);
        }
      });
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
    },

    changeCommentStatus(ids) {
      if (typeof ids == "boolean" && !this.multipleSelection.length) {
        return this.$message({
          message: "请先选中需要操作的用户",
          type: "error",
        });
      }
      let id =
        typeof ids == "boolean"
          ? this.multipleSelection.map((item) => item.id)
          : ids;
      let allText = typeof ids == "boolean" ? "选中" : "当前";
      this.$confirm("是否要禁止" + allText + "用户评论?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then((action) => {
        this.listLoading = true;
        changeCommentStatus({
          id,
          status: 0,
        }).then((res) => {
          this.$message({
            message: "操作成功",
            type: "success",
          });
          this.$refs.multipleTable.clearSelection();
          this.listLoading = false;
        });
      });
    },

    changeAccessStatus(ids) {
      if (typeof ids == "bollean" && !this.multipleSelection.length) {
        return this.$message({
          message: "请先选中需要操作的用户",
          type: "error",
        });
      }
      let id =
        typeof ids == "boolean"
          ? this.multipleSelection.map((item) => item.id)
          : ids;
      let allText = typeof ids == "boolean" ? "选中" : "当前";
      this.$confirm("是否要禁止" + allText + "用户访问？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then((action) => {
        (this.listLoading = true),
          changeAccessStatus({
            id,
            status: 0,
          }).then((res) => {
            this.$message({
              message: "操作成功",
              type: "success",
            });
            this.$refs.multipleTable.clearSelection();
            this.listLoading = false;
          });
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
