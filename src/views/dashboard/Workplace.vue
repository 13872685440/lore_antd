<template>
  <div>
    <page-view :title="false">
      <a-layout id="components-layout-demo-custom-trigger">
        <a-layout>
          <!-- 内容开始 -->
          <div class="gutter-example">
            <a-row :gutter="16">
              <a-col class="gutter-row" :xs="{ span: 24 }" :lg="{ span: 9 }">
                <div class="gutter-box personal" style="height: 110px">
                  <a-avatar
                    :size="84"
                    icon="user"
                    :src="avatar"
                    style="margin: 0 15px 15px 0; float: left"
                  />
                  <p>{{ timeFix }}，{{ user.name }}</p>
                  <template v-for="post in user.posts">
                    <span :key="post.organId" style="padding-right: 15px">
                      <span style="padding-right: 5px">
                        {{ post.organName }}
                      </span>
                      <span v-if="post.post_names">
                        {{ post.post_names }}
                        <a-divider />
                        <!-- <div class="personalContent">
                          <template v-for="names in post.post_names">
                            {{ names }}
                          </template>
                        </div>-->
                      </span>
                    </span>
                  </template>
                </div>

                <a-card :loading="remind_loading" :bordered="false">
                  <span slot="title">
                    <span>提醒</span>
                    <a-badge :count="remind_totalCount_x">
                      <span>&#160;&#160;&#160;&#160;</span>
                    </a-badge>
                  </span>
                  <a-list
                    itemLayout="vertical"
                    size="small"
                    :pagination="remind_pagination"
                    :dataSource="remind_data"
                    ref="_remind"
                  >
                    <a-list-item
                      slot="renderItem"
                      slot-scope="item"
                      key="item.id"
                    >
                      <a-list-item-meta>
                        <span slot="title" style="font-size: 14px">
                          <div
                            style="
                              width: 80%;
                              float: left;
                              padding-bottom: 10px;
                            "
                          >
                            <span v-if="item.r_type == '任务超时'">
                              您有一条
                              <span style="color: red"> {{ item.r_type }} </span
                              >的提醒，请尽快处理！
                            </span>
                            <span v-else-if="item.r_type == '抄送'">
                              您有一条
                              <a
                                style="color: #ffa500"
                                @click="showDrawer_tx(item)"
                              >
                                {{ item.r_type }}
                              </a>
                              提醒，请查看！
                            </span>
                          </div>
                          <div
                            style="
                              width: 20%;
                              float: left;
                              text-align: right;
                              padding-bottom: 10px;
                            "
                          >
                            <a-badge
                              v-if="
                                item.r_type == '抄送' && item.r_state == '0'
                              "
                              count="未读"
                              :number-style="{ backgroundColor: red }"
                            />
                            <a-badge
                              v-if="
                                item.r_type == '任务超时' &&
                                item.task.taskType != '已办'
                              "
                              count="待办理"
                              :number-style="{ backgroundColor: '#ff9900' }"
                            />
                          </div>
                        </span>
                        <span slot="description">
                          <div style="width: 50%; float: left">
                            {{ item.task.task_name }}
                          </div>
                          <div
                            style="width: 50%; float: left; text-align: right"
                          >
                            {{ item.txsj }}
                          </div>
                        </span>
                      </a-list-item-meta>
                    </a-list-item>
                  </a-list>
                </a-card>
              </a-col>
              <a-col
                class="gutter-row notice-box"
                :xs="{ span: 24 }"
                :lg="{ span: 15 }"
              >
                <div class="gutter-box">
                  <div class="h-title">
                    待办任务
                    <a-badge :count="task_totalCount">
                      <span>&#160;&#160;&#160;&#160;</span>
                    </a-badge>
                  </div>
                  <a-list
                    itemLayout="vertical"
                    size="small"
                    :pagination="pagination"
                    :dataSource="task_data"
                    ref="_task"
                  >
                    <a-list-item
                      slot="renderItem"
                      slot-scope="item"
                      key="item.id"
                    >
                      <template slot="actions">
                        <span>
                          <span
                            v-if="item.taskType === '待领取'"
                            style="color: #69aa46; cursor: pointer"
                            @click="receiveTask(item, 1)"
                          >
                            领取
                            <a-divider type="vertical" />
                          </span>
                          <span
                            v-if="item.taskType === '已领取'"
                            style="color: #ff892a; cursor: pointer"
                            @click="receiveTask(item, 2)"
                          >
                            取消
                            <a-divider type="vertical" />
                          </span>
                          <span
                            v-if="
                              item.taskType === '待办' ||
                              item.taskType === '已领取'
                            "
                          >
                            <a @click="showDrawer(item)" title="办理">办理</a>
                            <a-divider type="vertical" />
                          </span>
                          <span v-if="item.taskType === '挂起'">
                            <a @click="showDrawer(item)" title="办理"
                              >继续办理</a
                            >
                            <a-divider type="vertical" />
                          </span>
                          <!-- <span>
                            <a
                              @click="handle_view_image_c(item)"
                              title="流程图"
                              style="color: #a069c3"
                              >流程图</a
                            >
                          </span> -->
                        </span>
                      </template>
                      <a-list-item-meta>
                        <div slot="title">
                          <div
                            style="
                              width: 50%;
                              float: left;
                              padding-bottom: 10px;
                            "
                          >
                            {{ item.task_name }}
                          </div>
                          <div
                            style="
                              width: 50%;
                              float: left;
                              text-align: right;
                              padding-bottom: 10px;
                            "
                          >
                            <a-badge
                              :count="item.step_name"
                              :number-style="{ backgroundColor: '#52c41a' }"
                            />
                            <a-badge
                              v-if="item.time_out_type == '2'"
                              count="即将超时"
                              :number-style="{ backgroundColor: '#ff9900' }"
                            />
                            <a-badge
                              v-if="item.time_out_type == '1'"
                              count="超时"
                              :number-style="{ backgroundColor: red }"
                            />
                          </div>
                        </div>
                        <div slot="description">
                          <div style="width: 50%; float: left">
                            {{ item.previous_name }}
                          </div>
                          <div
                            style="width: 50%; float: left; text-align: right"
                          >
                            {{ item.ct }}
                          </div>
                        </div>
                      </a-list-item-meta>
                    </a-list-item>
                  </a-list>
                </div>
              </a-col>
            </a-row>
          </div>

          <!-- 内容结束 -->
        </a-layout>
      </a-layout>
    </page-view>
    <a-drawer
      width="85%"
      :title="draw_params.title"
      placement="right"
      :closable="draw_params.closable"
      :visible="draw_params.visible"
      :maskClosable="draw_params.maskClosable"
      :key="task_params.id"
      @close="toClose"
    >
      <d-form
        :form_params="form_params"
        :prop_params="prop_params"
        :task_params="task_params"
        :key="task_params.id"
        :task_view_params="task_view_params"
        :readOnly="true"
        ref="dform"
      ></d-form>
      <div
        :style="{
          position: 'absolute',
          bottom: 0,
          width: '100%',
          borderTop: '1px solid #e8e8e8',
          padding: '10px 16px',
          textAlign: 'right',
          left: 0,
          background: '#fff',
          borderRadius: '0 0 4px 4px',
        }"
      >
        <a-button @click="toClose">关闭</a-button>
        <a-button style="margin-left: 8px" @click="handle_view_image"
          >流程图</a-button
        >
        <a-button
          v-if="!form_params.readOnly"
          :key="task_params.id + 'a'"
          @click="submit_cxtj"
          type="primary"
          :style="{
            'margin-left': '8px',
          }"
          >重新提交</a-button
        >
        <template v-if="task_params.step_id == '00000000'"></template>
        <template v-for="b in step_buttons" v-else>
          <a-button
            v-if="b.color == null"
            :key="b.id"
            @click="submit(b)"
            :style="{
              'margin-left': '8px',
            }"
            >{{ b.name }}</a-button
          >
          <a-button
            v-else
            :style="{
              'margin-left': '8px',
              'background-color': b.color,
            }"
            @click="submit(b)"
            :key="b"
          >
            <span style="color: #fff">{{ b.name }}</span>
          </a-button>
        </template>
        <a-modal
          v-model="yj.showYj"
          ok-text="确认"
          cancel-text="取消"
          @ok="submit_yj"
        >
          <div slot="title">
            <span v-if="yj.showYj_type == '1'">意见</span>
            <span v-if="yj.showYj_type == '2'">
              意见
              <span style="color: red; padding-left: 5px">*</span>
            </span>
          </div>
          <a-textarea v-model="yj.shyj" />
        </a-modal>
        <modal-form-yj
          ref="modal_form"
          :hidden_datas="modal_hidden_datas"
          :init_params="modal_init_params"
          @ok="submit_yj_params($event)"
        ></modal-form-yj>
      </div>
    </a-drawer>
    <process-view
      :task_view_params="task_view_params"
      :key="form_params.query_params.id + task_view_params.current"
      ref="processview"
    ></process-view>
  </div>
</template>
<script>
import { timeFix } from "@/utils/util";
import { mixin, mixinDevice, main } from "@/utils/mixin";
import { PageView } from "@/layouts";
import { queryService, toService } from "@/api/manage";
import { mapActions } from "vuex";
import ProcessView from "@/components/CRUD/ProcessView.vue";
import Vue from "vue";
import ModalFormYj from "@/components/CRUD/ModalFormYj";
import { col_3_frist, col_2_all } from "@/utils/style";
import { MODEL_SYSTEM_DICTIONARY } from "@/store/mutation-types";
import axios from "axios";

export default {
  name: "Workplace",
  mixins: [mixin, mixinDevice, main],
  components: {
    PageView,
    ProcessView,
    ModalFormYj,
  },
  data() {
    return {
      timeFix: timeFix(),
      avatar: "",
      user: {},
      loading: true,
      path: {},
      task_data: [],
      task_totalCount: 0,
      pagination: {
        onChange: (page) => {
          console.log(page);
        },
        pageSize: 5,
      },
      remind_loading: true,
      remind_data: [],
      remind_totalCount: 0,
      remind_totalCount_x: 0,
      remind_pagination: {
        onChange: (page) => {
          console.log(page);
        },
        pageSize: 5,
      },
      yj: {
        showYj: false,
        showYj_type: "0",
        shyj: "",
        b: {},
      },
      a00010003: false,
      step_buttons: [],
      modal_form_datas: [],
      modal_hidden_datas: [],
      modal_init_params: {},
    };
  },
  computed: {
    userInfo() {
      return this.$store.getters.userInfo;
    },
  },
  created() {
    this.isChrome();
    const prefix = "/" + this.GLOBAL.MODEL_SYSTEM;
    this.user = this.userInfo;
    this.avatar = this.userInfo.avatar;
    this.path.task_path = prefix + "/taskext/main";
    this.path.remind_path = prefix + "/remind/main";
    this.path.remind_x_path = prefix + "/message/remind";
    this.path.receive_path = prefix + "/taskext/receive";
    this.path.cancelreceive_path = prefix + "/taskext/cancelreceive";
    this.path.init_button_path = prefix + "/processstep/init_button";
    this.task_view_params.open_task = true;
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.iniTask();
      vm.iniRemind();
    });
  },
  methods: {
    add(record) {
      var data = {};
      if (record != "") {
        data = record;
      }
      this.$refs.modal_form.add(data);
    },
    async receiveTask(item, i) {
      const data = { id: item.id };
      if (i == 1) {
        await toService("POST", this.path.receive_path, data).then((res) => {
          if (res.code === "400") {
            this.$notification.error({
              message: "任务领取失败",
              description: "任务已被领取",
            });
          } else if (res.code === "200") {
            this.$notification.success({
              message: "任务领取成功",
            });
            this.iniTask();
          }
        });
      } else {
        await toService("POST", this.path.cancelreceive_path, data).then(
          (res) => {
            if (res.code === "200") {
              this.$notification.success({
                message: "取消领取成功",
              });
              this.iniTask();
            }
          }
        );
      }
    },
    async iniTask() {
      await queryService(this.path.task_path, { pageSize: -1 }).then((res) => {
        this.task_data = res.result.data;
        this.task_totalCount = res.result.totalCount;
        if (res.result.totalCount <= 5) {
          this.pagination = false;
        }
        this.loading = false;
      });
    },
    async iniRemind() {
      const that = this;
      await axios
        .all([
          queryService(this.path.remind_path, { pageSize: -1 }),
          queryService(this.path.remind_x_path, {}),
        ])
        .then(
          axios.spread(function (r1, r2) {
            if (r1.code === "400" || r2.code === "400") {
              that.$notification.error({
                message: "参数错误",
                description: "请求出现错误，请联系管理员",
              });
            } else {
              that.remind_data = r1.result.data;
              that.remind_totalCount = r1.result.totalCount;
              if (r1.result.totalCount <= 5) {
                that.remind_pagination = false;
              }
              that.remind_totalCount_x = r2.result.badge;
              that.remind_loading = false;
            }
          })
        );
    },
    handle_view_image() {
      const that = this.$refs.dform;
      that.handle_view_image();
    },
    async handle_view_image_c(item) {
      var path = "";
      if (item.step_id.startsWith("1000")) {
        path = "/" + this.GLOBAL.MODEL_SYSTEM + "/wuzicaigou_process/init_step";
      } else if (item.step_id.startsWith("0001")) {
        path = "/" + this.GLOBAL.MODEL_SYSTEM + "/scouting_process/init_step";
      } else if (item.step_id.startsWith("0002")) {
        path =
          "/" + this.GLOBAL.MODEL_SYSTEM + "/qingshiinfo_process/init_step";
      } else if (item.step_id.startsWith("0003")) {
        path = "/" + this.GLOBAL.MODEL_SYSTEM + "/pjtinfo_process/init_step";
      } else if (item.step_id.startsWith("0004")) {
        path =
          "/" + this.GLOBAL.MODEL_SYSTEM + "/kaigongling_process/init_step";
      } else if (item.step_id.startsWith("0005")) {
        path = "/" + this.GLOBAL.MODEL_SYSTEM + "/pjtinfo_process/init_step";
      } else if (item.step_id.startsWith("0006")) {
        path = "/" + this.GLOBAL.MODEL_SYSTEM + "/cltinstall_process/init_step";
      } else if (item.step_id.startsWith("0007")) {
        path =
          "/" + this.GLOBAL.MODEL_SYSTEM + "/personinstall_process/init_step";
      } else if (item.step_id.startsWith("0008")) {
        path = "/" + this.GLOBAL.MODEL_SYSTEM + "/repairinfo_process/init_step";
      } else if (item.step_id.startsWith("0009")) {
        path =
          "/" + this.GLOBAL.MODEL_SYSTEM + "/importantwork_process/init_step";
      }
      await queryService(path, {
        entity_id: item.key_value,
        step_id: item.step_id,
      }).then((res) => {
        if (res.code === "400") {
          this.$notification.error({
            message: "参数错误",
            description: "请求出现错误，请联系管理员",
          });
        } else {
          this.task_view_params.prcocess_steps = res.result.prcocess_steps;
          this.task_view_params.comments = res.result.comments;
          this.task_view_params.title = res.result.title;
          this.task_view_params.current = res.result.current;
          this.task_view_params.comment_current = res.result.comment_current;
        }
      });
      const that = this.$refs.processview;
      that.showDrawer();
    },
    async submit_cxtj() {
      const that = this.$refs.dform;
      var params = { maps: {} };
      params.biaozhi = "1";
      that.handle_submit_cxtj(params).then((val) => {
        if (val == 1) {
          this.onClose();
          this.iniRemind();
          this.iniTask();
        }
      });
    },
    async showDrawer(item) {
      // this.entity_params.key_value = item.key_value;
      if (
        item.params_map.component_path != null &&
        item.params_map.component_path != ""
      ) {
        Vue.component("DForm", () =>
          Promise.resolve(
            require(`@/views${item.params_map.component_path}.vue`).default
          )
        );

        // if else 针对有特殊的，可以自己写逻辑 用if else
        await queryService(
          this.get_path(
            item.params_map.service_path +
              "/" +
              item.params_map.service_path_type
          ),
          { id: item.key_value }
        ).then((res) => {
          if (res.code === "400") {
            this.$notification.error({
              message: "参数错误",
              description: "请求出现错误，请联系管理员",
            });
          } else {
            this.edit_workspace({ task: item, entity: res.result });
          }
        });
        // 根据step_id渲染button
        if (!item.step_id.endsWith("0000")) {
          await queryService(this.path.init_button_path, {
            step_id: item.step_id,
          }).then((res) => {
            this.step_buttons = res.result;
          });
        } else {
          this.step_buttons = [];
        }
        if (item.step_id.startsWith("1000")) {
          this.path.submit_step_path = this.get_path(
            "/wuzicaigou_process/submit_step"
          );
        } else if (item.step_id.startsWith("0001")) {
          this.path.submit_step_path = this.get_path("/scouting/submit_step");
        }
        this.form_params.source = "workspace";
      }
    },
    async showDrawer_tx(itemx) {
      // this.entity_params.key_value = item.key_value;
      var item = itemx.task;
      if (
        item.params_map.component_path != null &&
        item.params_map.component_path != ""
      ) {
        Vue.component("DForm", () =>
          Promise.resolve(
            require(`@/views${item.params_map.component_path}.vue`).default
          )
        );

        // if else 针对有特殊的，可以自己写逻辑 用if else
        await queryService(
          this.get_path(
            item.params_map.service_path +
              "/" +
              item.params_map.service_path_type
          ),
          { id: item.key_value }
        ).then((res) => {
          if (res.code === "400") {
            this.$notification.error({
              message: "参数错误",
              description: "请求出现错误，请联系管理员",
            });
          } else {
            this.edit_workspace({ task: item, entity: res.result });
          }
        });
        if (itemx.r_type == "抄送") {
          // 根据step_id渲染button
          this.step_buttons = [];
          await toService("post", this.get_path("/remind/save"), {
            id: itemx.id,
            r_type: itemx.r_type,
            r_state: itemx.r_state,
          });
        }
        this.form_params.source = "workspace";
      }
    },
    async submit(b) {
      this.yj.b = b;
       if (b.showYj == "1" || b.showYj == "2") {
        this.yj.showYj_type = b.showYj;
        this.yj.showYj = true;
        this.yj.shyj = "";
      } else {
        this.submit_yj();
      }
    },
    submit_yj_params(params) {
      this.yj.b.task_id = this.task_params.id;
      this.yj.b.entity_id = this.form_params.query_params.id;
      this.yj.b.shyj = this.yj.shyj;
      this.yj.b.sqr_id = this.$store.getters.userId;

      //if (params.hqr_s != null) {
      // const hqrs = params.hqr_s.join(",");
      // alert(hqrs);
      //  }
      this.yj.b.maps = params;

      const that = this.$refs.dform;

      that.handle_submit_next(this.yj.b).then((val) => {
        if (val.code == 0) {
          this.$notification.error({
            message: "提交失败",
            description: val.message,
          });
        } else {
          this.iniTask();
          this.iniRemind();
          this.yj.showYj = false;
          this.draw_params.visible = false;
          this.$notification.success({
            message: "提交成功",
            description: val.message,
          });
        }
      });
    },
    async submit_yj() {
      if (this.yj.b.showYj == "2") {
        if (this.yj.shyj == null || this.yj.shyj == "") {
          this.$message.warn("请填写意见");
          return;
        }
      }
      this.yj.b.task_id = this.task_params.id;
      this.yj.b.entity_id = this.form_params.query_params.id;
      this.yj.b.shyj = this.yj.shyj;
      this.yj.b.sqr_id = this.$store.getters.userId;
      this.yj.b.maps = {};
      const that = this.$refs.dform;

      that.handle_submit_next(this.yj.b).then((val) => {
        if (val.code == 0) {
          this.$notification.error({
            message: "提交失败",
            description: val.message,
          });
        } else {
          this.iniTask();
          this.iniRemind();
          this.yj.showYj = false;
          this.draw_params.visible = false;
          this.$notification.success({
            message: "提交成功",
            description: val.message,
          });
        }
      });
    },
    toClose() {
      this.iniRemind();
      this.onClose();
    },
    //  async submit_cxtj() {
    // const that = this.$refs.dform.$refs.rform;

    //  submit_cxtj(
    //    that,
    //    this.task_params.id,
    //    this.task_params.task_id,
    //    this.$refs.dform.form_prop_params.save_path
    //  ).then((val) => {
    //    if (val == 1) {
    //     this.onClose();
    //      this.iniTask();
    //   }
    //  });
    //  },
    isChrome() {
      var u = navigator.userAgent;
      var isChrome = u.indexOf("Chrome") > -1; // 是否是谷歌
      if (!this.isMobile()) {
        if (!isChrome) {
          this.handleLogout();
        }
      }
    },
    ...mapActions(["Logout"]),
    isMobile() {
      let flag = navigator.userAgent.match(
        /(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i
      );
      return flag;
    },

    handleLogout() {
      const that = this;
      this.$confirm({
        title: "提示",
        content: "请使用谷歌浏览器",
        onOk() {
          return that
            .Logout({})
            .then(() => {
              window.location.reload();
            })
            .catch((err) => {
              that.$message.error({
                title: "错误",
                description: err.message,
              });
            });
        },
        onCancel() {
          return that
            .Logout({})
            .then(() => {
              window.location.reload();
            })
            .catch((err) => {
              that.$message.error({
                title: "错误",
                description: err.message,
              });
            });
        },
      });
    },
  },
};
</script>
<style>
.gutter-box .notice-content {
  color: rgba(0, 0, 0, 0.45);
}
.gutter-box .notice-content:hover {
  color: #1890ff;
}
.gutter-box .ant-list-item-action {
  /* margin-left: 20px; */
}
.home-box {
  margin: 12px 12px 0px 12px;
  padding: 14px 24px;
  /* min-height: 280px; */
  background-color: #ffffff;
}
.notice-box {
  /* padding: 14px 24px; */
}
.basics {
  background: linear-gradient(
    to right,
    rgb(172 239 230 / 90%),
    rgb(130 201 225 / 90%)
  );
  padding: 15px 24px;
}
.basics h1 {
  font-size: 30px;
  font-weight: 100;
  margin-bottom: 0;
}
.basics p {
  font-size: 12px;
  color: rgb(20 118 85);
}

.h-title {
  border-bottom: 1px solid #e8e8e8;
  padding-bottom: 10px;
}

.employee .employee-box {
  text-align: center;
}
.employee .employee-box img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  margin-top: 20px;
}
.employee .employee-box p {
  line-height: 30px;
}
.gutter-example {
  padding: 1px 12px 0 12px;
}
.gutter-example >>> .ant-row > div {
  background: transparent;
  border: 0;
}
.gutter-box {
  background: #ffffff;
  padding: 14px 24px;
  margin-bottom: 14px;
  border-radius: 5px;
  box-shadow: rgb(171 163 163 / 55%) 2px 4px 8px 0px;
}
#components-layout-demo-custom-trigger .trigger {
  font-size: 18px;
  line-height: 64px;
  padding: 0 24px;
  cursor: pointer;
  transition: color 0.3s;
}

#components-layout-demo-custom-trigger .trigger:hover {
  color: #1890ff;
}

#components-layout-demo-custom-trigger .logo {
  height: 32px;
  background: linear-gradient(
    to right,
    rgb(222 233 252 / 90%),
    rgb(175 232 252 / 90%)
  );
  margin: 16px;
}
/* 任务完成率 */
.statistics {
  padding-bottom: 30px;
}

.statistics p {
  margin: 10px 0 0 0;
}

.info .lf {
  float: left;
  height: inherit;
  width: 50%;
  text-align: center;
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
}
.info .rg {
  float: left;
  width: 50%;
  text-align: center;
}
.info .rg h2 {
  margin-bottom: 0px;
  margin-top: 12px;
}
.info .rg p {
  font-size: 12px;
  color: #929292;
}
.info .gutter-box {
  padding: 0;
  height: 80px;
}
.gutter-box .notice-i {
  float: right;
  font-style: normal;
  color: rgb(0 0 0);
  font-size: 10px;
  font-weight: 100;
}
.ant-list-item-action > li {
  cursor: auto !important;
}
.ant-drawer-body {
  padding-bottom: 80px;
}
</style>
