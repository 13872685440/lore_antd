<template>
  <a-spin :spinning="spinning">
    <a-form :form="form" class="form">
      <base-form
        :needId="init_params.needId"
        v-if="init_params.needBase"
        :hidden_datas="hidden_datas"
        :needProcess="init_params.needProcess"
      >
      </base-form>
      <!--  <r-form :form_datas="form_datas"> </r-form>-->
      <template v-for="(item, i) in form_datas">
        <a-form-item
          :label="item.label"
          :labelCol="item.style_col != null ? item.style_col.labelCol : ''"
          :wrapperCol="item.style_col != null ? item.style_col.wrapperCol : ''"
          :key="item.prop"
          v-show="item.show == null ? true : item.show"
          v-if="item.type === 'Slot'"
        >
          <slot :name="item.prop" :data="item" :form="form"></slot>
        </a-form-item>
        <r-form
          :item="item"
          :key="i"
          :tmpId="tmpId"
          :readOnly="init_params.readOnly"
          v-else
        >
        </r-form>
      </template>
      <slot name="forms" :form="form"></slot>
    </a-form>
    <process-view
      :task_view_params="task_view_params"
      :key="form_params.query_params.id + task_view_params.current"
      ref="processview"
    ></process-view>
  </a-spin>
</template>
<script>
import { editForm } from "@/utils/mixin.js";
import BaseForm from "@/components/CRUD/BaseForm.vue";
import { queryService } from "@/api/manage";
import RForm from "./RForm.vue";
import ProcessView from "./ProcessView.vue";

export default {
  mixins: [editForm],
  components: { BaseForm, RForm, ProcessView },
  name: "Form_Base",
  props: {
    // 渲染服务路径
    edit_path: {
      type: String,
    },
    // 传递的form参数
    form_params: {
      type: Object,
    },
    // 流程参数
    task_params: { type: Object },
    // 实体值
    entity_datas: {
      type: Object,
    },
    // 隐藏参数
    hidden_datas: {
      type: Array,
      default: function () {
        return [];
      },
    },
    // form表单参数
    form_datas: {
      type: Array,
    },
  },
  data() {
    return {
      task_view_params: {
        visible: false,
      },
      form: this.$form.createForm(this),
    };
  },

  async created() {
    await this.init();
  },
  mounted() {},
  methods: {
    async init() {
      this.spinning = true;

      // 以下是初始化参数，当传递时取传递的值，否则取默认值
      // 标题
      this.init_params.title = this.assign(
        this.init_params.title,
        this.form_params.title
      );
      // 是否需要card包裹，和标题结合使用 默认=false
      this.init_params.needCard = this.assign(
        this.init_params.needCard,
        this.form_params.needCard
      );
      // 是否需要默认渲染id 当id是assigned时（即code模式）时，需要设置该值=true 默认=false
      this.init_params.needId = this.assign(
        this.init_params.needId,
        this.form_params.needId
      );
      // 是否需要流程 当启用流程时 需要设置该值=true 默认=false
      this.init_params.needProcess = this.assign(
        this.init_params.needProcess,
        this.form_params.needProcess
      );
      // 是否需要baseform baseform中是baseentity的属性的渲染 默认=true
      this.init_params.needBase = this.assign(
        this.init_params.needBase,
        this.form_params.needBase
      );
      // 是否需要重载form 当我们在操作form时 且不跳转页面时 需要设置该值=true 默认=false
      this.init_params.reRender = this.assign(
        this.init_params.reRender,
        this.form_params.reRender
      );
      // 是否只读 默认=false
      this.init_params.readOnly = this.assign(
        this.init_params.readOnly,
        this.form_params.readOnly
      );

      // 渲染时的参数，通常我们都是通过id作为参数进行渲染
      this.init_params.query_params = this.assign(
        this.init_params.query_params,
        this.form_params.query_params
      );

      // tmpId tmpId主要是为了上传附件时使用
      if (
        this.form_params.query_params != null &&
        this.form_params.query_params.id != null &&
        this.form_params.query_params.id != ""
      ) {
        this.tmpId = this.form_params.query_params.id;
      }

      // 当传递实体值时，直接渲染实体值 最常见的使用方法是：直接通过list将实体值进行传递，这样可以避免去后台再去查询，节省效率
      // 否则，将会通过服务路径去后台进行查询
      // 服务路径传递时，取传递的路径，否则使用默认路径 默认路径为菜单的path+"/edit"
      if (this.entity_datas != null) {
        this.form.setFieldsValue(this.entity_datas);
        this.spinning = false;
      } else {
        if (this.form_params.edit_path != null) {
          this.path.edit_path = this.form_params.edit_path;
        } else {
          this.path.edit_path = this.get_path(this.$route.path, "/edit");
        }
        await queryService(
          this.path.edit_path,
          this.form_params.query_params
        ).then((res) => {
          this.form.setFieldsValue(res.result);
          this.spinning = false;
        });
      }

      // 保存路径 默认路径为菜单的path+"/save"
      if (this.form_params.save_path != null) {
        this.path.save_path = this.form_params.save_path;
      } else {
        this.path.save_path = this.get_path(this.$route.path, "/save");
      }
      // 提交路径 默认路径为菜单的"/processstep/submit_process"
      if (this.form_params.submit_path != null) {
        this.path.submit_path = this.form_params.submit_path;
      } else {
        this.path.submit_path = this.get_path(
          "/processstep",
          "/submit_process"
        );
      }

      // 初始化流程参数
      if (
        this.task_params != null &&
        this.task_params.open_task != null &&
        this.task_params.open_task
      ) {
        // 初始化流程步骤参数 默认路径为"/processstep/init_step"
        if (this.task_params.inistep_path != null) {
          this.path.inistep_path = this.task_params.inistep_path;
        } else {
          this.path.inistep_path = this.get_path("/processstep", "/init_step");
        }

        var params = {};
        if (
          this.task_params.task_id != null &&
          this.task_params.task_id != ""
        ) {
          params.task_id = this.task_params.task_id;
        } else {
          params.app_id = this.task_params.app_id;
        }
        if (
          this.form_params.query_params != null &&
          this.form_params.query_params.id != null &&
          this.form_params.query_params.id != ""
        ) {
          // 当实体已经持久化时，则通过task_id 和 entity_id进行查询
          params.entity_id = this.form_params.query_params.id;
        }

        await queryService(this.path.inistep_path, params).then((res) => {
          if (res.code === "400") {
            this.$notification.error({
              message: "参数错误",
              description: "请求出现错误，请联系管理员",
            });
          } else {
            this.task_view_params.steps = res.result.steps;
            this.task_view_params.title = res.result.title;
            this.task_view_params.current = res.result.current;
          }
        });
      }
    },
    showDrawer() {
      this.task_view_params.visible = true;
    },
    onClose() {
      this.task_view_params.visible = false;
    },
  },
};
</script>