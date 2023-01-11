<template>
  <y-form
    ref="rform"
    :form_params="form_params"
    :service_path="service_path"
    :needProcess="needProcess"
    :form_datas="form_datas"
    :hidden_datas="hidden_datas"
    :params="params"
    :task_view_params="task_view_params"
  >
    <a-select
      slot="lore_type"
      size="default"
      mode="default"
      placeholder="请选择知识类型"
      :disabled="form_params.readOnly"
      v-decorator="[
        'lore_type',
        { rules: [{ required: true, message: '请选择知识类型' }] },
      ]"
    >
      <a-select-option
        v-for="type in loretypes"
        :key="type.id"
        :value="type.id"
        >{{ type.name }}</a-select-option
      >
    </a-select>
    <quill-editor
      slot="content_editor"
      class="editor"
      ref="myTextEditor"
      v-model="content_editor"
      :options="editorOption"
      @change="onEditorChange($event)"
    >
    </quill-editor>
  </y-form>
</template>
<script>
import YForm from "@/components/CRUD/YForm.vue";
import {
  queryService,
  save,
  submit,
  submit_cxtj,
  saveService,
} from "@/api/manage";
import { main } from "@/utils/mixin";

export default {
  components: { YForm },
  name: "Form_Base",
  mixins: [main],

  props: {
    // 打开form时传递的参数
    form_params: {
      type: Object,
    },
    // 属性参数,用来加载公共属性，如list，tree等
    prop_params: {
      type: Object,
    },
    // 流程属性
    task_view_params: { type: Object },
  },
  data() {
    return {
      service_path: {},
      needProcess: true,
      params: {
        xmystz: 0,
        tmpId: "",
      },
      loretypes: [],
      hidden_datas: ["sqr_id", "content", "org_id"],
      form_datas: [
        {
          label: "标题",
          prop: "title",
          rules: [
            {
              required: true,
              message: "请输入标题",
              whitespace: true,
            },
          ],
          type: "Input",
        },
        {
          label: "关键字",
          prop: "keyword",
          rules: [
            {
              required: true,
              message: "请输入关键字",
              whitespace: true,
            },
          ],
          type: "Input",
        },
        {
          label: "知识类型",
          prop: "lore_type",
          type: "Slot",
        },
        {
          label: "有效期",
          prop: "expiry_date",
          rules: [{ required: true, message: "请选择有效期" }],
          type: "Date",
          format: "YYYY-MM-DD",
        },
        {
          label: "积分",
          prop: "score",
          rules: [
            {
              required: true,
              message: "请输入积分",
             },
          ],
          type: "Input_Number",
        },
        {
          label: "内容",
          prop: "content_editor",
          type: "Slot",
        },
        {
          label: "上传文件",
          prop: "a1",
          type: "UploadFile",
          datas: {
            ebcn: "com.cat.lore.model.Lore",
            sign: "A1",
          },
        },
      ],
      content_editor: null,
      editorOption: {
        modules: {
          toolbar: [
            ["bold", "italic", "underline", "strike"], // 加粗 斜体 下划线 删除线
            ["blockquote", "code-block"], // 引用  代码块
            [{ header: 1 }, { header: 2 }], // 1、2 级标题
            [{ list: "ordered" }, { list: "bullet" }], // 有序、无序列表
            [{ script: "sub" }, { script: "super" }], // 上标/下标
            [{ indent: "-1" }, { indent: "+1" }], // 缩进
            // [{'direction': 'rtl'}],                         // 文本方向
            [{ size: ["small", false, "large", "huge"] }], // 字体大小
            [{ header: [1, 2, 3, 4, 5, 6, false] }], // 标题
            [{ color: [] }, { background: [] }], // 字体颜色、字体背景颜色
            [{ font: [] }], // 字体种类
            [{ align: [] }], // 对齐方式
            ["clean"], // 清除文本格式
            ["link", "image", "video"], // 链接、图片、视频
          ], //工具菜单栏配置
        },
        placeholder: "请在这里添加知识内容或在下方上传文件", //提示
        readyOnly: false, //是否只读
        theme: "snow", //主题 snow/bubble
        syntax: true, //语法检测
      },
    };
  },
  created() {
    this.params.tmpId = this.form_params.query_params.tmp_id;
        console.log("this.form_params.readOnlsssssssssssssssssssssssssssy");

    console.log(this.form_params.readOnly);
    this.init();
    const prefix = "/" + this.GLOBAL.MODEL_SYSTEM;
    this.service_path.edit_path = prefix + "/lore/edit";
    this.service_path.save_path = prefix + "/lore/save";
    this.service_path.submit_path = prefix + "/lore_process/submit_process";
    this.service_path.submit_next_path = prefix + "/lore_process/submit_step";
    this.service_path.init_step_path = prefix + "/lore_process/init_step";
  },
  async mounted() {
    queryService(this.get_path("/loretype/types"), {}).then((res) => {
      if (res.code === "400") {
        this.$notification.error({
          message: "参数错误",
          description: "请求出现错误，请联系管理员",
        });
      } else {
        this.loretypes = res.result;
      }
      this.spinning = false;
    });
  },
  computed: {
    editor() {
      return this.$refs.myTextEditor.quillEditor;
    },
  },
  methods: {
    onEditorChange(editor) {
      this.content_editor = editor.html;
    },
    init() {
      this.content_editor = this.form_params.entity_datas.content;
    },
    handle_save() {
      return new Promise((resolve) => {
        const that = this.$refs.rform;
        that.form.setFieldsValue({
          content: this.content_editor,
        });
        save(that).then((val) => {
          resolve(val);
        });
      });
    },
    handle_submit(params) {
      const that = this.$refs.rform;
      console.log(that.form.getFieldValue("org_id"));
      params.organ_id = that.form.getFieldValue("org_id");
      return new Promise((resolve) => {
        submit(that, params).then((val) => {
          resolve(val);
        });
      });
    },
    handle_submit_cxtj(params) {
      const that = this.$refs.rform;
      params.organ_id = that.form.getFieldValue("org_id");
      //params.maps.fzr_id = that.form.getFieldValue("fzr_id");
      params.step_id = that.form.getFieldValue("step_id");
      params.maps.sgdwsh = that.form.getFieldValue("sgdwsh");
      return new Promise((resolve) => {
        submit_cxtj(that, params).then((val) => {
          resolve(val);
        });
      });
    },
    handle_submit_next(params) {
      const that = this.$refs.rform;
      params.organ_id = that.form.getFieldValue("org_id");
      return new Promise((resolve) => {
        saveService(this.service_path.submit_next_path, params).then((res) => {
          if (res.code != "200") {
            resolve({ code: 0, message: res.message });
          } else {
            resolve({ code: 1, message: res.message });
          }
        });
      });
    },
    async handle_view_image() {
      console.log("1111111111111111111111");
      const that = this.$refs.rform;
      await queryService(this.service_path.init_step_path, {
        step_id:
          that.form.getFieldValue("step_id") == null
            ? "0001"
            : that.form.getFieldValue("step_id"),
        entity_id: this.form_params.query_params.id,
      }).then((res) => {
        console.log(res);
        if (res.code === "400") {
          this.$notification.error({
            message: "参数错误",
            description: "请求出现错误，请联系管理员",
          });
        } else {
          //this.task_view_params.open_task = true;
          this.task_view_params.prcocess_steps = res.result.prcocess_steps;
          this.task_view_params.comments = res.result.comments;
          this.task_view_params.title = res.result.title;
          this.task_view_params.current = res.result.current;
          this.task_view_params.comment_current = res.result.comment_current;
        }
      });
      const thatx = that.$refs.processview;
      thatx.showDrawer();
    },
  },
};
</script>