<template>
  <div class="meedu-main-body">
    <div class="float-left j-b-flex mb-30">
      <div class="d-flex">
        <p-button
          text="添加学员"
          p="member.store"
          @click="$router.push({ name: 'MemberCreate' })"
          type="primary"
        >
        </p-button>
        <p-button
          text="批量导入"
          p="member.import"
          @click="$router.push({ name: 'MemberImport' })"
          type="primary"
          class="ml-15"
        >
        </p-button>
      </div>
      <div class="d-flex">
        <div>
          <el-input
            class="w-150px"
            v-model="filter.keywords"
            placeholder="学员关键字"
          ></el-input>
        </div>
        <div class="ml-10">
          <el-date-picker
            :picker-options="pickerOptions"
            v-model="filter.created_at"
            type="daterange"
            align="right"
            unlink-panels
            range-separator="至"
            start-placeholder="注册开始日期"
            end-placeholder="注册结束日期"
          >
          </el-date-picker>
        </div>
        <div class="ml-10">
          <el-button @click="paginationReset">清空</el-button>
          <el-button @click="firstPageLoad" type="primary">筛选</el-button>
        </div>
        <div class="drawerMore d-flex ml-10" @click="drawer = true">
          <template v-if="showStatus">
            <img src="../../assets/img/icon-filter-h.png" />
            <span class="act">已选</span>
          </template>
          <template v-else>
            <img src="../../assets/img/icon-filter.png" />
            <span>更多</span>
          </template>
        </div>
      </div>
    </div>
    <div class="float-left" v-loading="loading">
      <div class="float-left">
        <el-table
          :header-cell-style="{ background: '#f1f2f9' }"
          :data="users"
          class="float-left"
          @sort-change="sortChange"
          :default-sort="{ prop: 'id', order: 'descending' }"
        >
          <el-table-column prop="id" sortable label="学员ID" width="100">
          </el-table-column>
          <el-table-column label="学员" width="210">
            <template slot-scope="scope">
              <div class="user-item">
                <div class="avatar">
                  <img :src="scope.row.avatar" width="40" height="40" />
                </div>
                <div class="nickname">{{ scope.row.nick_name }}</div>
              </div>
            </template>
          </el-table-column>
          <el-table-column sortable="" label="注册时间" width="250">
            <template slot-scope="scope">{{
              scope.row.created_at | dateFormat
            }}</template>
          </el-table-column>
          <el-table-column prop="mobile" label="手机号" width="180">
          </el-table-column>
          <el-table-column label="VIP会员" width="150">
            <template slot-scope="scope">
              <span v-if="scope.row.role">{{ scope.row.role.name }}</span>
              <span v-else></span>
            </template>
          </el-table-column>
          <el-table-column prop="credit1" sortable label="积分" width="120">
          </el-table-column>
          <el-table-column label="标签" width="150">
            <template slot-scope="scope">
              <el-tag
                class="ml-5"
                v-for="(item, index) in scope.row.tags"
                :key="index"
              >
                {{ item.name }}
              </el-tag>
            </template>
          </el-table-column>

          <el-table-column label="备注">
            <template slot-scope="scope">
              <div
                v-if="userRemark[scope.row.id]"
                v-html="userRemark[scope.row.id].remark"
              ></div>
            </template>
          </el-table-column>

          <el-table-column fixed="right" label="操作" width="100">
            <template slot-scope="scope">
              <p-link
                text="查看"
                p="member.detail"
                type="primary"
                @click="detail(scope.row)"
              ></p-link>
              <p-link
                class="ml-5"
                text="发消息"
                p="member.message.send"
                type="primary"
                @click="sendMessage(scope.row)"
              ></p-link>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <div class="float-left mt-30 text-center">
        <el-pagination
          @size-change="paginationSizeChange"
          @current-change="paginationPageChange"
          :current-page="pagination.page"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pagination.size"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        >
        </el-pagination>
      </div>
    </div>
    <el-drawer :size="360" :visible.sync="drawer" :with-header="false">
      <div class="n-padding-box">
        <div class="tit flex">更多筛选</div>
        <div class="j-flex">
          <el-input
            class="w-300px"
            v-model="filter.keywords"
            placeholder="学员列表关键字"
          ></el-input>
        </div>
        <div class="j-flex mt-20">
          <el-select
            v-model="filter.role_id"
            class="w-300px"
            placeholder="VIP会员"
            filterable
          >
            <el-option
              v-for="(item, index) in filterData.roles"
              :key="index"
              :label="item.name"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </div>

        <div class="j-flex mt-20">
          <el-select
            v-model="filter.tag_id"
            class="w-300px"
            placeholder="学员标签"
            filterable
          >
            <el-option
              v-for="(item, index) in filterData.tags"
              :key="index"
              :label="item.name"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </div>
        <div class="j-flex mt-20">
          <el-date-picker
            :picker-options="pickerOptions"
            v-model="filter.created_at"
            type="daterange"
            align="right"
            unlink-panels
            range-separator="至"
            start-placeholder="注册开始日期"
            end-placeholder="注册结束日期"
          >
          </el-date-picker>
        </div>
        <div class="j-b-flex mt-30">
          <el-button @click="paginationReset">清空</el-button>
          <el-button @click="firstPageLoad" type="primary">筛选</el-button>
        </div>
      </div>
    </el-drawer>
    <el-dialog title="发消息" :visible.sync="visible" width="400px">
      <div class="d-flex">
        <el-input
          type="textarea"
          maxlength="500"
          resize="none"
          show-word-limit
          v-model="message"
          placeholder="请输入消息文本"
          class="w-100"
          rows="4"
        ></el-input>
      </div>
      <div class="j-r-flex mt-20">
        <el-button @click="confirm" type="primary">确认</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pageName: "member-list",
      pagination: {
        page: 1,
        size: 10,
        sort: "id",
        order: "desc",
      },
      filter: {
        keywords: null,
        role_id: null,
        tag_id: null,
        created_at: null,
      },
      total: 0,
      loading: false,
      users: [],
      userRemark: [],
      filterData: {
        tags: [],
        roles: [],
      },
      drawer: false,
      showStatus: false,
      pickerOptions: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
      },
      visible: false,
      message: null,
      mid: null,
    };
  },
  activated() {
    this.getUser();
    this.$utils.scrollTopSet(this.pageName);
  },
  beforeRouteLeave(to, from, next) {
    this.$utils.scrollTopRecord(this.pageName);
    next();
  },
  watch: {
    "filter.role_id"(val) {
      if (val) {
        this.showStatus = true;
      } else {
        this.showStatus = false;
      }
    },
    "filter.tag_id"(val) {
      if (val) {
        this.showStatus = true;
      } else {
        this.showStatus = false;
      }
    },
    "filter.keywords"(val) {
      if (val) {
        this.showStatus = true;
      } else {
        this.showStatus = false;
      }
    },
    "filter.created_at"(val) {
      if (val) {
        this.showStatus = true;
      } else {
        this.showStatus = false;
      }
    },
  },
  methods: {
    paginationReset() {
      this.pagination.page = 1;
      this.filter.keywords = null;
      this.filter.role_id = null;
      this.filter.tag_id = null;
      this.filter.created_at = null;
      this.getUser();
      this.drawer = false;
    },
    paginationSizeChange(size) {
      this.pagination.size = size;
      this.getUser();
    },
    paginationPageChange(page) {
      this.pagination.page = page;
      this.getUser();
    },
    firstPageLoad() {
      this.pagination.page = 1;
      this.getUser();
      this.drawer = false;
    },
    sortChange(column) {
      this.pagination.sort = column.prop;
      this.pagination.order = column.order === "ascending" ? "asc" : "desc";
      this.getUser();
    },
    getUser() {
      if (this.loading) {
        return;
      }
      this.loading = true;
      let params = {};
      Object.assign(params, this.filter);
      Object.assign(params, this.pagination);
      this.$api.Member.List(params).then((res) => {
        this.loading = false;
        this.users = res.data.data.data;
        this.total = res.data.data.total;
        // 学员备注
        this.userRemark = res.data.user_remarks;

        this.filterData.tags = res.data.tags;
        this.filterData.roles = res.data.roles;
      });
    },
    detail(item) {
      this.$router.push({ name: "MemberDetail", params: { userId: item.id } });
    },
    sendMessage(item) {
      this.visible = true;
      this.mid = item.id;
    },
    confirm() {
      this.$api.Member.SendMessage(this.mid, {
        message: this.message,
      }).then((res) => {
        this.$message.success(this.$t("common.success"));
        this.message = null;
        this.mid = null;
        this.visible = false;
        this.getUser();
      });
    },
  },
};
</script>

<style lang="less" scoped>
.user-item {
  width: auto;
  display: flex;
  align-items: center;
  .avatar {
    margin-right: 10px;
  }
  .nickname {
    font-size: 15px;
    font-weight: normal;
  }
}
</style>
