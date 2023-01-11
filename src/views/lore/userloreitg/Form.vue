<template>
  <y-form
    ref="rform"
    :form_params="form_params"
    :service_path="service_path"
    :needProcess="needProcess"
    :form_datas="form_datas"
    :params="params"
  >
    <template slot="forms">
      <a-card size="small">
        <span slot="title"> 物资采购（领用）计划清单 </span>
        <a-table
          :columns="modal_columns"
          :data-source="datas"
          :pagination="pagination"
        >
        </a-table>
      </a-card>
    </template>
  </y-form>
</template>
<script>
import YForm from "@/components/CRUD/YForm.vue";
import { queryService } from "@/api/manage";

export default {
  components: { YForm },
  name: "Form_Base",
  props: {
    // 打开form时传递的参数
    form_params: {
      type: Object,
    },
    // 属性参数,用来加载公共属性，如list，tree等
    prop_params: {
      type: Object,
    },
  },
  data() {
    return {
      service_path: {},
      needProcess: false,
      form_datas: [
        {
          label: "姓名",
          prop: "user_name",
          type: "Input",
        },
        {
          label: "积分",
          prop: "integral",
          type: "Input",
          readOnly: true,
        },
      ],
      modal_columns: [
        {
          title: "知识",
          dataIndex: "lore_name",
          key: "lore_name",
        },
        {
          title: "积分",
          dataIndex: "integral",
          key: "integral",
        },
      ],
      modal_form_params: {
        query_path: "/" + this.GLOBAL.MODEL_SYSTEM + "/useritgrcd/main",
      },
      datas: [],
      pagination: {
        size: 10,
      },
    };
  },
  created() {
    this.params.tmpId = this.form_params.query_params.tmp_id;
    this.init();
    const prefix = "/" + this.GLOBAL.MODEL_SYSTEM;
    this.service_path.edit_path = prefix + "/userloreitg/edit";
  },
  mounted() {},
  methods: {
    async init() {
      if (
        this.form_params.query_params.id != null &&
        this.form_params.query_params.id != ""
      ) {
        this.initAppPers(this.form_params.query_params.id);
      } else {
        this.initAppPers(this.form_params.query_params.tmpId);
      }
    },
    async initAppPers() {
      await queryService(this.modal_form_params.query_path, {
        pageSize: -1,
      }).then((res) => {
        this.datas = res.result.data;
        if (res.result.totalCount <= 10) {
          this.pagination = false;
        }
      });
    },
  },
};
</script>