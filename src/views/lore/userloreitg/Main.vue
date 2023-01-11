<template>
  <div>
    <s-list
      :list_columns="list_columns"
      :list_query_params="list_query_params"
      ref="slist"
    >
      <span slot="oper_action" slot-scope="record">
        <a @click="edit({ record: record.data, readOnly: false })" title="编辑">
          <a-icon type="edit" style="color: #69aa46" />
        </a>
      </span>
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
        ref="sform"
      >
      </s-form>

      <div class="a-folat">
        <a-spin :spinning="spinning">
          <a-button @click="onClose">关闭</a-button>
        </a-spin>
      </div>
    </a-drawer>
  </div>
</template>

<script>
import SList from "@/components/CRUD/SList.vue";
import SForm from "./Form";
import { main } from "@/utils/mixin";
import store from "@/store";

export default {
  name: "Main_pjtinfo",
  mixins: [main],
  components: {
    SList,
    SForm,
  },
  data() {
    return {
      list_query_params: [{ name: "姓名", code: "user_name", type: "Input" }],
      // list 列参数
      list_columns: [
        {
          title: "姓名",
          dataIndex: "user_name",
          sorter: true,
        },
        {
          title: "部门",
          dataIndex: "organ_name",
          sorter: true,
        },
        {
          title: "积分",
          dataIndex: "integral",
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
      this.draw_params.title = "用户积分";
      this.form_params.reRender = true;
      this.form_params.needProcess = false;
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

