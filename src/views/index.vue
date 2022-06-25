<template>
  <div class="components-container">
    <el-row :gutter="10">
      <el-col :span="6">
        <div class="bg-purple">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span class="box-card-title">总公告数</span>
              <!-- <el-button style="float: right; padding: 3px 0" type="text">
                操作按钮
              </el-button> -->
            </div>
            <div class="">
              {{ noticeTotal }}
            </div>
          </el-card>
        </div>
      </el-col>
      <el-col :span="6"
        ><div class="bg-purple">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span class="box-card-title">总代办数</span>
            </div>
            <div class="">
              {{ todoTotal }}
            </div>
          </el-card>
        </div>
      </el-col>
      <el-col :span="6"
        ><div class="bg-purple">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span class="box-card-title">总已办数</span>
            </div>
            <div class=" ">
              {{ finishedTotal }}
            </div>
          </el-card>
        </div>
      </el-col>
      <el-col :span="6"
        ><div class="bg-purple">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span class="box-card-title">总文件数</span>
            </div>
            <div class=" ">
              {{ fileTotal }}
            </div>
          </el-card>
        </div>
      </el-col>
    </el-row>
    <el-row :gutter="10" style="margin-top: -10px">
      <el-col :span="24">
        <div class="grid-content bg-purple">
          <template>
            <el-tabs v-model="activeName" @tab-click="handleClick">
              <el-tab-pane label="通知公告" name="first">
                <el-form
                  :model="queryParams"
                  ref="queryForm"
                  size="small"
                  :inline="true"
                  v-show="showSearch"
                  label-width="68px"
                >
                  <el-form-item label="标题" prop="noticeTitle">
                    <el-input
                      v-model="queryParams.noticeTitle"
                      placeholder="请输入标题"
                      clearable
                      @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="操作人员" prop="createBy">
                    <el-input
                      v-model="queryParams.createBy"
                      placeholder="请输入操作人员"
                      clearable
                      @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="类型" prop="noticeType">
                    <el-select
                      v-model="queryParams.noticeType"
                      placeholder="类型"
                      clearable
                    >
                      <el-option
                        v-for="dict in dict.type.sys_notice_type"
                        :key="dict.value"
                        :label="dict.label"
                        :value="dict.value"
                      />
                    </el-select>
                  </el-form-item>
                  <el-form-item>
                    <el-button
                      type="primary"
                      icon="el-icon-search"
                      size="mini"
                      @click="handleQuery"
                      >搜索</el-button
                    >
                    <el-button
                      icon="el-icon-refresh"
                      size="mini"
                      @click="resetQuery"
                      >重置</el-button
                    >
                  </el-form-item>
                </el-form>
                <el-table v-loading="loading" :data="noticeList">
                  <el-table-column
                    label="序号"
                    align="center"
                    prop="noticeId"
                    width="100"
                  />
                  <el-table-column
                    label="标题"
                    align="center"
                    prop="noticeTitle"
                    :show-overflow-tooltip="true"
                  />
                  <el-table-column
                    label="类型"
                    align="center"
                    prop="noticeType"
                    width="100"
                  >
                    <template slot-scope="scope">
                      <dict-tag
                        :options="dict.type.sys_notice_type"
                        :value="scope.row.noticeType"
                      />
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="状态"
                    align="center"
                    prop="status"
                    width="100"
                  >
                    <template slot-scope="scope">
                      <dict-tag
                        :options="dict.type.sys_notice_status"
                        :value="scope.row.status"
                      />
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="创建者"
                    align="center"
                    prop="createBy"
                    width="100"
                  />
                  <el-table-column
                    label="创建时间"
                    align="center"
                    prop="createTime"
                    width="100"
                  >
                    <template slot-scope="scope">
                      <span>{{
                        parseTime(scope.row.createTime, "{y}-{m}-{d}")
                      }}</span>
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="操作"
                    align="center"
                    class-name="small-padding fixed-width"
                  >
                    <template slot-scope="scope">
                      <el-button
                        size="mini"
                        type="text"
                        @click="handleLook(scope.row)"
                        v-hasPermi="['system:notice:edit']"
                        >查看</el-button
                      >
                    </template>
                  </el-table-column>
                </el-table>

                <pagination
                  v-show="noticeTotal > 0"
                  :total="noticeTotal"
                  :page.sync="queryParams.pageNum"
                  :limit.sync="queryParams.pageSize"
                  @pagination="getList"
                />
                <!-- 查看公告 -->
                <el-dialog
                  :title="title"
                  :visible.sync="open"
                  width="780px"
                  append-to-body
                >
                  <div class="notice">
                    <div class="notice-title">{{ form.noticeTitle }}</div>
                    <div
                      class="notice-content"
                      v-html="form.noticeContent"
                    ></div>
                  </div>
                </el-dialog>
              </el-tab-pane>
              <!-- 我的待办 -->
              <el-tab-pane label="我的待办" name="second">
                <el-table v-loading="loading" :data="todoList">
                  <el-table-column
                    label="任务编号"
                    align="center"
                    prop="taskId"
                    :show-overflow-tooltip="true"
                  />
                  <el-table-column
                    label="流程名称"
                    align="center"
                    prop="procDefName"
                  />
                  <el-table-column
                    label="任务节点"
                    align="center"
                    prop="taskName"
                  />
                  <el-table-column label="流程版本" align="center">
                    <template slot-scope="scope">
                      <el-tag size="medium"
                        >v{{ scope.row.procDefVersion }}</el-tag
                      >
                    </template>
                  </el-table-column>
                  <el-table-column label="流程发起人" align="center">
                    <template slot-scope="scope">
                      <label
                        >{{ scope.row.startUserName }}
                        <el-tag type="info" size="mini">{{
                          scope.row.startDeptName
                        }}</el-tag></label
                      >
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="接收时间"
                    align="center"
                    prop="createTime"
                    width="180"
                  />
                  <el-table-column
                    label="操作"
                    align="center"
                    class-name="small-padding fixed-width"
                  >
                    <template slot-scope="scope">
                      <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-edit-outline"
                        @click="handleProcess(scope.row)"
                        >处理
                      </el-button>
                    </template>
                  </el-table-column>
                </el-table>

                <pagination
                  v-show="todoTotal > 0"
                  :total="todoTotal"
                  :page.sync="queryParams.pageNum"
                  :limit.sync="queryParams.pageSize"
                  @pagination="getList"
                />
              </el-tab-pane>
              <!-- 我的已办 -->
              <el-tab-pane label="我的已办" name="third">
                <el-table v-loading="loading" :data="finishedList">
                  <el-table-column
                    label="任务编号"
                    align="center"
                    prop="taskId"
                    :show-overflow-tooltip="true"
                  />
                  <el-table-column
                    label="流程名称"
                    align="center"
                    prop="procDefName"
                    :show-overflow-tooltip="true"
                  />
                  <el-table-column
                    label="任务节点"
                    align="center"
                    prop="taskName"
                  />
                  <el-table-column label="流程发起人" align="center">
                    <template slot-scope="scope">
                      <label
                        >{{ scope.row.startUserName }}
                        <el-tag type="info" size="mini">{{
                          scope.row.startDeptName
                        }}</el-tag></label
                      >
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="接收时间"
                    align="center"
                    prop="createTime"
                    width="180"
                  />
                  <el-table-column
                    label="审批时间"
                    align="center"
                    prop="finishTime"
                    width="180"
                  />
                  <el-table-column
                    label="耗时"
                    align="center"
                    prop="duration"
                    width="180"
                  />
                  <el-table-column
                    label="操作"
                    align="center"
                    class-name="small-padding fixed-width"
                  >
                    <template slot-scope="scope">
                      <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-tickets"
                        @click="handleFlowRecord(scope.row)"
                        >流转记录</el-button
                      >
                      <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-tickets"
                        @click="handleRevoke(scope.row)"
                        >撤回
                      </el-button>
                    </template>
                  </el-table-column>
                </el-table>

                <pagination
                  v-show="finishedTotal > 0"
                  :total="finishedTotal"
                  :page.sync="queryParams.pageNum"
                  :limit.sync="queryParams.pageSize"
                  @pagination="getList"
                />
              </el-tab-pane>
              <!-- 文件资料 -->
              <el-tab-pane label="文件资料" name="four">
                <el-form
                  :model="queryParams"
                  ref="queryForm"
                  :inline="true"
                  v-show="showSearch"
                  label-width="68px"
                >
                  <el-form-item label="文件名" prop="fileName">
                    <el-input
                      v-model="queryParams.fileName"
                      placeholder="请输入文件名"
                      clearable
                      size="small"
                      @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="原名" prop="originalName">
                    <el-input
                      v-model="queryParams.originalName"
                      placeholder="请输入原名"
                      clearable
                      size="small"
                      @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="文件后缀" prop="fileSuffix">
                    <el-input
                      v-model="queryParams.fileSuffix"
                      placeholder="请输入文件后缀"
                      clearable
                      size="small"
                      @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="创建时间">
                    <el-date-picker
                      v-model="daterangeCreateTime"
                      size="small"
                      style="width: 240px"
                      value-format="yyyy-MM-dd"
                      type="daterange"
                      range-separator="-"
                      start-placeholder="开始日期"
                      end-placeholder="结束日期"
                    ></el-date-picker>
                  </el-form-item>
                  <el-form-item label="上传人" prop="createBy">
                    <el-input
                      v-model="queryParams.createBy"
                      placeholder="请输入上传人"
                      clearable
                      size="small"
                      @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="服务商" prop="service">
                    <el-input
                      v-model="queryParams.service"
                      placeholder="请输入服务商"
                      clearable
                      size="small"
                      @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item>
                    <el-button
                      type="primary"
                      icon="el-icon-search"
                      size="mini"
                      @click="handleQuery"
                      >搜索</el-button
                    >
                    <el-button
                      icon="el-icon-refresh"
                      size="mini"
                      @click="resetQuery"
                      >重置</el-button
                    >
                  </el-form-item>
                </el-form>

                <el-table v-loading="loading" :data="ossList">
                  <el-table-column type="selection" width="55" align="center" />
                  <el-table-column
                    label="对象存储主键"
                    align="center"
                    prop="ossId"
                    v-if="false"
                  />
                  <el-table-column
                    label="文件名"
                    align="center"
                    prop="fileName"
                  />
                  <el-table-column
                    label="原名"
                    align="center"
                    prop="originalName"
                  />
                  <el-table-column
                    label="文件后缀"
                    align="center"
                    prop="fileSuffix"
                  />
                  <el-table-column label="文件展示" align="center" prop="url">
                    <template slot-scope="scope">
                      <el-image
                        v-if="
                          previewListResource &&
                          checkFileSuffix(scope.row.fileSuffix)
                        "
                        style="width: 100px; height: 100px"
                        :src="scope.row.url"
                        :preview-src-list="[scope.row.url]"
                      />
                      <span
                        v-text="scope.row.url"
                        v-if="
                          !checkFileSuffix(scope.row.fileSuffix) ||
                          !previewListResource
                        "
                      />
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="创建时间"
                    align="center"
                    prop="createTime"
                    width="180"
                  >
                    <template slot-scope="scope">
                      <span>{{
                        parseTime(scope.row.createTime, "{y}-{m}-{d}")
                      }}</span>
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="上传人"
                    align="center"
                    prop="createBy"
                  />
                  <el-table-column
                    label="服务商"
                    align="center"
                    prop="service"
                  />
                  <el-table-column
                    label="操作"
                    align="center"
                    class-name="small-padding fixed-width"
                  >
                    <template slot-scope="scope">
                      <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-edit"
                        @click="handleDownload(scope.row)"
                        v-hasPermi="['system:oss:download']"
                        >下载</el-button
                      >
                    </template>
                  </el-table-column>
                </el-table>

                <pagination
                  v-show="fileTotal > 0"
                  :total="fileTotal"
                  :page.sync="queryParams.pageNum"
                  :limit.sync="queryParams.pageSize"
                  @pagination="getList"
                />
              </el-tab-pane>
            </el-tabs>
          </template>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { listNotice, getNotice } from "@/api/system/notice";
import { listTodoProcess } from "@/api/workflow/process";
import { listFinishedProcess } from "@/api/workflow/process";
import { revokeProcess } from "@/api/workflow/finished";
import { listOss } from "@/api/system/oss";
export default {
  name: "",
  components: {},
  dicts: ["sys_notice_status", "sys_notice_type"],
  data() {
    return {
      activeName: "first",
      // 按钮loading
      buttonLoading: false,
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 公告总条数
      noticeTotal: 0,
      // 流程待办任务表格数据
      todoList: [],
      // 待办总条数
      todoTotal: 0,
      // 总文件条数
      fileTotal: 0,
      // OSS对象存储表格数据
      ossList: [],
      // 预览列表图片
      previewListResource: true,
      // 创建时间时间范围
      daterangeCreateTime: [],
      // 公告表格数据
      noticeList: [],
      // 弹出层标题
      title: "",
      // 显示搜索条件
      showSearch: true,
      // 总已办条数
      finishedTotal: 0,
      // 已办任务列表数据
      finishedList: [],
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        noticeTitle: undefined,
        createBy: undefined,
        status: undefined,
        name: null,
        category: null,
        key: null,
        tenantId: null,
        deployTime: null,
        derivedFrom: null,
        derivedFromRoot: null,
        parentDeploymentId: null,
        engineVersion: null,
        fileName: undefined,
        originalName: undefined,
        fileSuffix: undefined,
        url: undefined,
        createTime: undefined,
        createBy: undefined,
        service: undefined,
      },
      // 表单参数
      form: {},
    };
  },
  created() {
    this.getList();
  },
  methods: {
    handleClick(tab, event) {
      console.log(tab, event);
    },
    getList() {
      // 查询公告列表
      this.loading = true;
      listNotice(this.queryParams).then((response) => {
        this.noticeList = response.rows;
        this.noticeTotal = response.total;
        this.loading = false;
      });
      // 查询代办
      listTodoProcess(this.queryParams).then((response) => {
        this.todoList = response.rows;
        this.todoTotal = response.total;
        this.loading = false;
      });
      // 查询已办
      listFinishedProcess(this.queryParams).then((response) => {
        this.finishedList = response.rows;
        this.finishedTotal = response.total;
        this.loading = false;
      });
      // 查询文件
      this.queryParams.params = {};
      if (null != this.daterangeCreateTime && "" != this.daterangeCreateTime) {
        this.queryParams.params["beginCreateTime"] =
          this.daterangeCreateTime[0];
        this.queryParams.params["endCreateTime"] = this.daterangeCreateTime[1];
      }
      this.getConfigKey("sys.oss.previewListResource").then((response) => {
        this.previewListResource =
          response.msg === undefined ? true : response.msg === "true";
      });
      listOss(this.queryParams).then((response) => {
        this.ossList = response.rows;
        this.fileTotal = response.total;
        this.loading = false;
      });
    },
    // 查看公告
    handleLook(row) {
      const noticeId = row.noticeId || this.ids;
      getNotice(noticeId).then((response) => {
        this.form = response.data;
        this.open = true;
        this.title = "公告内容";
      });
    },
    // 搜索按钮操作
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },

    // 重置按钮操作
    resetQuery() {
      this.daterangeCreateTime = [];
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 跳转到处理页面
    handleProcess(row) {
      this.$router.push({
        path: "/work/detail",
        query: {
          definitionId: row.procDefId,
          procInsId: row.procInsId,
          deployId: row.deployId,
          taskId: row.taskId,
          finished: true,
        },
      });
    },
    // 流程流转记录
    handleFlowRecord(row) {
      this.$router.push({
        path: "/work/detail",
        query: {
          definitionId: row.procDefId,
          procInsId: row.procInsId,
          deployId: row.deployId,
          taskId: row.taskId,
          finished: false,
        },
      });
    },
    // 撤回任务
    handleRevoke(row) {
      const params = {
        instanceId: row.procInsId,
      };
      revokeProcess(params).then((res) => {
        this.$modal.msgSuccess(res.msg);
        this.getList();
      });
    },
    // 任务日志列表查询
    handleOssConfig() {
      this.$router.push({ path: "/system/oss-config/index" });
    },
    // 下载按钮操作
    handleDownload(row) {
      this.$download.oss(row.ossId);
    },
    checkFileSuffix(fileSuffix) {
      let arr = ["png", "jpg", "jpeg"];
      return arr.some((type) => {
        return fileSuffix.indexOf(type) > -1;
      });
    },
  },
};
</script>

<style rel="stylesheet/scss" lang="scss">
.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}

.box-card {
  width: 100%;
}
.box-card-title {
  font-size: 14px;
}
.el-row {
  margin-bottom: 20px;
  &:last-child {
    margin-bottom: 0;
  }
}
.el-col {
  border-radius: 3px;
}
.bg-purple {
  background: #ffffff;
}
.grid-content {
  border-radius: 3px;
  min-height: 36px;
  padding: 5px 20px;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  border-radius: 4px;
  border: 1px solid #e6ebf5;
  background-color: #ffffff;
  overflow: hidden;
  color: #303133;
  -webkit-transition: 0.3s;
  transition: 0.3s;
}
.row-bg {
  padding: 10px 0;
  background-color: #ffffff;
}
.notice {
  font-size: 14px;
}
.notice-title {
  width: 100%;
  text-align: center;
}
.notice-content {
  width: 100%;
  text-indent: 2em;
}
</style>
