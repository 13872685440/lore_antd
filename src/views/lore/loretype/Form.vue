<template>
  <x-form
    ref="rform"
    :form_params="form_params"
    :form_datas="form_datas"
    :service_path="service_path"
  >
  </x-form>
</template>
<script>
import XForm from "@/components/CRUD/XForm.vue";
import { col_3_frist, col_3_last,col_2_frist } from "@/utils/style";
export default {
  components: { XForm },
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
      form_datas: [
        [
          {
            label: "上级编码",
            prop: "sc_id",
            type: "Input",
            readOnly: true,
            styles: col_3_frist,
          },
          {
            label: "本级编码",
            prop: "clc",
            type: "Input",
            readOnly: true,
            styles: col_3_last,
          },
          {
            label: "完整编码",
            prop: "id",
            type: "Input",
            readOnly: true,
            styles: col_3_last,
          },
        ],
        [
          {
            label: "上级类型",
            prop: "scName",
            type: "Input",
            readOnly: true,
            styles: col_3_frist,
          },
          {
            label: "类型名称",
            prop: "name",
            type: "Input",
            rules: [
              { required: true, message: "请输入类型名称", whitespace: true },
            ],
            styles: col_3_last,
          },
          {
            label: "排序",
            prop: "xh",
            type: "Input_Number",
            rules: [
              { type: "number", message: "请输入数字", whitespace: true },
            ],
            styles: col_3_last,
          },
        ],
        [
         {
            label: "可见部门",
            prop: "organ_id",
            rules: [{ required: true, message: "请选择" }],
            type: "Tree_Simple",
            datas: this.prop_params.organs,
            styles: col_2_frist,
          }, 
        ],
      ],
    };
  },
  created() {
    const prefix = "/" + this.GLOBAL.MODEL_SYSTEM;
    if (this.form_params.type == "addlower") {
      this.service_path.edit_path = prefix + "/loretype/addlower";
    } else {
      this.service_path.edit_path = prefix + "/loretype/edit";
    }
    this.service_path.save_path = prefix + "/loretype/save";
  },
  mounted() {},
  methods: {},
};
</script>