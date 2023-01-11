<template>
  <div>
    <s-list
      :list_columns="list_columns"
      :list_query_params="list_query_params"
      ref="slist"
    >
      <span slot="ac_button">
        <a-button type="primary" icon="plus" @click="edit_new()"
          >新建</a-button
        ></span
      >
      <span slot="oper_action" slot-scope="record">
        <a @click="edit({ record: record.data, readOnly: false })" title="编辑">
          <a-icon type="edit" style="color: #69aa46" />
        </a>
        <a-divider type="vertical" />
        <a @click="handle_delete(record.data.id)" v-if="record.data.canEdit">
          <a-icon type="delete" style="color: #ff6347" title="删除" /> </a
      ></span>
    </s-list>
    <a-drawer
      width="85%"
      :title="draw_params.title"
      placement="right"
      :closable="draw_params.closable"
      :visible="draw_params.visible"
      :maskClosable="draw_params.maskClosable"
      :key="form_params.query_params.tmp_id"
      @close="onClose"
      ><s-form
        :prop_params="prop_params"
        :form_params="form_params"
        :key="form_params.query_params.tmp_id + form_params.readOnly"
        :task_view_params="task_view_params"
        ref="sform"
      >
      </s-form>

      <div class="a-folat">
        <a-spin :spinning="spinning">
          <a-button @click="onClose">关闭</a-button>
          <!-- <a-button style="margin-left: 8px" @click="handle_view_image"
            >流程图</a-button
          > -->

          <a-button
            style="margin-left: 8px"
            @click="handle_save"
            type="primary"
            v-if="!form_params.readOnly"
            >保存</a-button
          ><template
            v-if="
              !form_params.readOnly &&
              form_params.query_params.id != null &&
              form_params.query_params.id != ''
            "
          >
            <a-button
              style="margin-left: 8px"
              @click="handle_submit"
              type="danger"
              >提交</a-button
            ></template
          >
        </a-spin>
      </div>
    </a-drawer>
  </div>
</template>

<script>
import SList from "@/components/CRUD/SList.vue";
import SForm from "./Form";
import { main } from "@/utils/mixin";

export default {
  name: "Main_pjtinfo",
  mixins: [main],
  components: {
    SList,
    SForm,
  },
  data() {
    return {
      list_query_params: [
        { name: "标题", code: "title", type: "Input" },
        { name: "关键字", code: "keyword", type: "Input" },
      ],
      // list 列参数
      list_columns: [
        {
          title: "标题",
          dataIndex: "title",
          sorter: true,
        },
        {
          title: "关键字",
          dataIndex: "keyword",
        },
        {
          title: "类型",
          dataIndex: "lore_type_name",
        },
        {
          title: "有效期",
          dataIndex: "expiry_date",
        },
        {
          title: "积分",
          dataIndex: "score",
        },

        {
          title: "流程状态",
          dataIndex: "step_name",
          scopedSlots: { customRender: "lczt" },
        },
        {
          title: "申请时间",
          dataIndex: "ct",
        },
        {
          title: "操作",
          dataIndex: "action",
          width: "150px",
          scopedSlots: { customRender: "action" },
        },
      ],
    };
  },
  async created() {
    this.init();
  },
  methods: {
    // 初始化方法
    init() {
      this.draw_params.title = "知识库";
      this.form_params.reRender = true;
      this.task_view_params.open_task = true;
      this.form_params.needProcess = true;
    },
    handle_save() {
      this.spinning = true;
      const that = this.$refs.sform;
      that.handle_save().then((val) => {
        if (val == 1) {
          this.onClose();
          this.$refs.slist.refreshTable();
        } else if (val == 2) {
          this.$refs.slist.refreshTable();
        }
        this.form_params.query_params.id =
          that.$refs.rform.form.getFieldValue("id");
        this.spinning = false;
      });
    },
    handle_submit() {
      this.spinning = true;
      const that = this.$refs.sform;
      var params = { maps: {} };
      params.biaozhi = "0";
      params.app_id = this.task_params.app_id;
      that.handle_submit(params).then((val) => {
        if (val == 1) {
          this.onClose();
          this.$refs.slist.refreshTable();
        }
        this.spinning = false;
      });
    },
    handle_submit_cxtj() {
      this.spinning = true;
      const that = this.$refs.sform;
      var params = { maps: {} };
      params.biaozhi = "1";
      that.handle_submit_cxtj(params).then((val) => {
        if (val == 1) {
          this.onClose();
          this.$refs.slist.refreshTable();
        }
        this.spinning = false;
      });
    },
    handle_delete(id) {
      this.$refs.slist.handle_delete(id);
    },
    handle_view_image() {
      const that = this.$refs.sform;
      that.handle_view_image();
    },
  },
};
</script>
<style>
.a-folat {
  position: absolute;
  bottom: 0;
  width: 100%;
  border-top: 1px solid #e8e8e8;
  padding: 10px 16px;
  text-align: right;
  left: 0;
  background: #fff;
  border-radius: 0 0 4px 4px;
}
.ant-drawer-body {
  padding-bottom: 80px;
}
</style>

