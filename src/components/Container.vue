<template>
  <el-container class="fm2-container" :class="{disabled: disabled}">
    <el-main class="fm2-main">
      <el-container>
        <el-aside width="250px">
          <div class="components-list">
            <template v-if="basicFields.length">
              <div class="widget-cate">{{$t('fm.components.basic.title')}}</div>
              <draggable
                tag="ul"
                :disabled="disabled"
                :list="allowedBasicComponents"
                v-bind="{group:{ name:'people', pull:'clone',put:false},sort:false, ghostClass: 'ghost'}"
                @end="handleMoveEnd"
                @start="handleMoveStart"
                :move="handleMove"
              >
                <li
                  class="form-edit-widget-label"
                  :class="{'no-put': item.type == 'divider'}"
                  v-for="(item, index) in allowedBasicComponents"
                  :key="index"
                >
                  <a>
                    <i class="icon iconfont" :class="item.icon"></i>
                    <span>{{item.name}}</span>
                  </a>
                </li>
              </draggable>
            </template>

            <template v-if="advanceFields.length">
              <div class="widget-cate">{{$t('fm.components.advance.title')}}</div>
              <draggable
                tag="ul"
                :disabled="disabled"
                :list="allowedAdvanceComponents"
                v-bind="{group:{ name:'people', pull:'clone',put:false},sort:false, ghostClass: 'ghost'}"
                @end="handleMoveEnd"
                @start="handleMoveStart"
                :move="handleMove"
              >
                <!-- v-if="advanceFields.indexOf(item.type) >= 0" -->
                <li
                  class="form-edit-widget-label"
                  :class="{'no-put': item.type == 'table'}"
                  v-for="(item, index) in allowedAdvanceComponents"
                  :key="index"
                >
                  <a>
                    <i class="icon iconfont" :class="item.icon"></i>
                    <span>{{item.name}}</span>
                  </a>
                </li>
              </draggable>
            </template>

            <template v-if="layoutFields.length">
              <div class="widget-cate">{{$t('fm.components.layout.title')}}</div>
              <draggable
                tag="ul"
                :disabled="disabled"
                :list="allowedLayoutComponents"
                v-bind="{group:{ name:'people', pull:'clone',put:false},sort:false, ghostClass: 'ghost'}"
                @end="handleMoveEnd"
                @start="handleMoveStart"
                :move="handleMove"
              >
                <li
                  class="form-edit-widget-label no-put"
                  v-for="(item, index) in allowedLayoutComponents"
                  :key="index"
                >
                  <a>
                    <i class="icon iconfont" :class="item.icon"></i>
                    <span>{{item.name}}</span>
                  </a>
                </li>
              </draggable>
            </template>
          </div>
        </el-aside>
        <el-container class="center-container" direction="vertical">
          <el-header class="btn-bar" style="height: 45px;">
            <slot name="action"></slot>

            <el-button
              v-if="clearable"
              type="text"
              :size="actionSize"
              icon="el-icon-delete"
              @click="handleClear"
            >{{$t('fm.actions.clear')}}</el-button>
            <el-button
              v-if="preview"
              type="text"
              :size="actionSize"
              icon="el-icon-view"
              @click="handlePreview"
            >{{$t('fm.actions.preview')}}</el-button>
            <el-button
              v-if="copyCode"
              type="text"
              :size="actionSize"
              icon="el-icon-document"
              @click="handleCopyCode(true)"
            >{{$t('fm.actions.copyCode')}}</el-button>
            <el-button
              v-if="generateJson"
              type="text"
              :size="actionSize"
              icon="el-icon-tickets"
              @click="handleGenerateJson"
            >{{$t('fm.actions.json')}}</el-button>
            <el-button
              v-if="upload"
              type="text"
              :size="actionSize"
              icon="el-icon-upload2"
              @click="handleUpload"
            >{{$t('fm.actions.import')}}</el-button>
          </el-header>
          <el-main :class="{'widget-empty': widgetForm.list.length == 0}">
            <widget-form
              v-if="!resetJson"
              ref="widgetForm"
              :data="widgetForm"
              :select.sync="widgetFormSelect"
            ></widget-form>
          </el-main>
        </el-container>

        <el-aside class="widget-config-container">
          <el-container>
            <el-header height="45px">
              <div
                class="config-tab"
                :class="{active: configTab=='widget'}"
                @click="handleConfigSelect('widget')"
              >{{$t('fm.config.widget.title')}}</div>
              <div
                class="config-tab"
                :class="{active: configTab=='form'}"
                @click="handleConfigSelect('form')"
              >{{$t('fm.config.form.title')}}</div>
            </el-header>
            <el-main class="config-content">
              <widget-config v-show="configTab=='widget'" :data="widgetFormSelect"></widget-config>
              <form-config v-show="configTab=='form'" :data="widgetForm.config"></form-config>
            </el-main>
          </el-container>
        </el-aside>
        <cus-dialog
          :visible="previewVisible"
          @on-close="previewVisible = false"
          ref="widgetPreview"
          width="100%"
          :fullscreen="true"
          form
        >
          <!-- 运行源码 -->
          <code-run ref="codeRunRef" :data="widgetForm" :value="widgetModels" :remote="remoteFuncs"></code-run>
          <template slot="action">
            <el-button type="info" @click="handleCopyCode">{{$t('fm.actions.copyCode')}}</el-button>
            <el-button type="primary" @click="handleTest">{{$t('fm.actions.getData')}}</el-button>
            <el-button @click="handleReset">{{$t('fm.actions.reset')}}</el-button>
          </template>
        </cus-dialog>

        <cus-dialog
          :visible="uploadVisible"
          @on-close="uploadVisible = false"
          @on-submit="handleUploadJson"
          ref="uploadJson"
          width="800px"
          form
        >
          <el-alert type="info" :title="$t('fm.description.uploadJsonInfo')"></el-alert>
          <div id="uploadeditor" style="height: 400px;width: 100%;">{{jsonEg}}</div>
        </cus-dialog>

        <cus-dialog
          :visible="jsonVisible"
          @on-close="jsonVisible = false"
          ref="jsonPreview"
          width="800px"
          form
        >
          <div id="jsoneditor" style="height: 400px;width: 100%;">{{jsonTemplate}}</div>

          <template slot="action">
            <el-button
              type="primary"
              class="json-btn"
              @click="copyJsonDataHandler"
            >{{$t('fm.actions.copyData')}}</el-button>
          </template>
        </cus-dialog>

        <cus-dialog
          :visible="codeVisible"
          @on-close="codeVisible = false"
          ref="codePreview"
          width="800px"
          form
          :action="false"
        >
          <!-- <div id="codeeditor" style="height: 500px; width: 100%;">{{htmlTemplate}}</div> -->
          <el-tabs type="border-card" style="box-shadow: none;" v-model="codeActiveName">
            <el-tab-pane label="Vue Component" name="vue">
              <div id="vuecodeeditor" style="height: 500px; width: 100%;">{{vueTemplate}}</div>
            </el-tab-pane>
            <el-tab-pane label="HTML" name="html">
              <div id="codeeditor" style="height: 500px; width: 100%;">{{htmlTemplate}}</div>
            </el-tab-pane>
          </el-tabs>
        </cus-dialog>
      </el-container>
    </el-main>
    <el-footer v-if="!hideFooter" height="30px" style="font-weight: 600;">
      Powered by
      <a
        target="_blank"
        href="https://github.com/giscafer/vue-form-builder"
      >vue-form-builder</a>
    </el-footer>
  </el-container>
</template>

<script>
import Draggable from "vuedraggable";
import WidgetConfig from "./WidgetConfig";
import FormConfig from "./FormConfig";
import WidgetForm from "./WidgetForm";
import CusDialog from "./CusDialog";
import GenerateForm from "./GenerateForm";
import {
  basicComponents,
  layoutComponents,
  advanceComponents
} from "./componentsConfig.js";
import { loadJs, loadCss } from "../util/index.js";
import request from "../util/request.js";
import genFormCode from "../generator/generateFormCode";
import copyText from "../util/copy";

import "brace/index";
import "brace/mode/json";
import "brace/mode/html";

export default {
  name: "fm-making-form",
  components: {
    Draggable,
    WidgetConfig,
    FormConfig,
    WidgetForm,
    CusDialog,
    GenerateForm
  },
  props: {
    actionSize: {
      type: String,
      default: "medium"
    },
    hideFooter: {
      type: Boolean,
      default: false
    },
    copyCode: {
      type: Boolean,
      default: false
    },
    preview: {
      type: Boolean,
      default: false
    },
    generateJson: {
      type: Boolean,
      default: false
    },
    upload: {
      type: Boolean,
      default: false
    },
    clearable: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    basicComponentsConfig: {
      type: Array,
      default: () => basicComponents,
    },
    advanceComponentsConfig: {
      type: Array,
      default: () => advanceComponents,
    },
    layoutComponentsConfig: {
      type: Array,
      default: () => layoutComponents,
    },
    basicFields: {
      type: Array,
      default: () => [
        "input",
        "textarea",
        "number",
        "radio",
        "checkbox",
        "time",
        "date",
        "rate",
        "color",
        "select",
        "switch",
        "slider",
        "text"
      ]
    },
    advanceFields: {
      type: Array,
      default: () => ["blank", "imgupload", "editor", "cascader"]
    },
    layoutFields: {
      type: Array,
      default: () => ["grid"]
    }
  },
  data() {
    return {
      basicComponents,
      layoutComponents,
      advanceComponents,
      resetJson: false,
      widgetForm: {
        list: [],
        config: {
          labelWidth: 100,
          labelPosition: "right",
          size: "small"
        }
      },
      configTab: "widget",
      widgetFormSelect: null,
      previewVisible: false,
      jsonVisible: false,
      codeVisible: false,
      uploadVisible: false,
      remoteFuncs: {
        func_test(resolve) {
          setTimeout(() => {
            const options = [
              { id: "1", name: "1111" },
              { id: "2", name: "2222" },
              { id: "3", name: "3333" }
            ];

            resolve(options);
          }, 2000);
        },
        funcGetToken(resolve) {
          request
            .get("http://tools-server.xiaoyaoji.cn/api/uptoken")
            .then(res => {
              resolve(res.uptoken);
            });
        },
        upload_callback(response, file, fileList) {
          console.log("callback", response, file, fileList);
        }
      },
      widgetModels: {},
      blank: "",
      htmlTemplate: "",
      vueTemplate: "",
      jsonTemplate: "",
      uploadEditor: null,
      jsonCopyValue: "",
      jsonClipboard: null,
      jsonEg: `{
  "list": [],
  "config": {
    "labelWidth": 100,
    "labelPosition": "top",
    "size": "small"
  }
}`,
      codeActiveName: "vue"
    };
  },
  computed: {
    allowedBasicComponents() {
      return this.basicComponents.filter(item => this.basicFields.indexOf(item.type)>=0)
    },
    allowedAdvanceComponents() {
      return this.advanceComponents.filter(item => this.advanceFields.indexOf(item.type)>=0)
    },
    allowedLayoutComponents() {
      return this.layoutComponents.filter(item => this.layoutFields.indexOf(item.type)>=0)
    },
  },
  mounted() {
    this._loadComponents();
  },
  methods: {
    _loadComponents() {
      this.basicComponents = this.basicComponentsConfig.map(item => {
        return {
          ...item,
          name: this.$t(`fm.components.fields.${item.type}`)
        };
      });
      this.advanceComponents = this.advanceComponentsConfig.map(item => {
        return {
          ...item,
          name: this.$t(`fm.components.fields.${item.type}`)
        };
      });
      this.layoutComponents = this.layoutComponentsConfig.map(item => {
        return {
          ...item,
          name: this.$t(`fm.components.fields.${item.type}`)
        };
      });
    },
    handleGoGithub() {
      window.location.href = "https://github.com/giscafer/vue-form-builder";
    },
    handleConfigSelect(value) {
      this.configTab = value;
    },
    handleMoveEnd(evt) {
      console.log("end", evt);
    },
    handleMoveStart({ oldIndex }) {
      console.log("start", oldIndex, this.basicComponents);
    },
    handleMove() {
      return true;
    },
    handlePreview() {
      // 没控制必须正则
      try {
        new RegExp(this.data.options.pattern);
      } catch (e) {
        console.log("无效正则");
      }
      console.log(this.widgetForm);
      this.previewVisible = true;
    },
    handleTest() {
      this.$refs.generateForm
        .getData()
        .then(data => {
          this.$alert(data, "").catch(e => {});
          this.$refs.widgetPreview.end();
        })
        .catch(e => {
          this.$refs.widgetPreview.end();
        });
    },
    handleReset() {
      this.$refs.generateForm.reset();
    },
    handleGenerateJson() {
      this.jsonVisible = true;
      this.jsonTemplate = this.widgetForm;
      console.log(JSON.stringify(this.widgetForm));
      this.$nextTick(() => {
        const editor = ace.edit("jsoneditor");
        editor.session.setMode("ace/mode/json");

        this.jsonCopyValue = JSON.stringify(this.widgetForm);
      });
    },
    copyJsonDataHandler() {
      const cpSuccess = copyText(this.jsonCopyValue);
      if (cpSuccess) {
        this.$message.success(this.$t("fm.message.copySuccess"));
      }
    },
    handleUpload() {
      this.uploadVisible = true;
      this.$nextTick(() => {
        this.uploadEditor = ace.edit("uploadeditor");
        this.uploadEditor.session.setMode("ace/mode/json");
      });
    },
    handleUploadJson() {
      try {
        this.setJSON(JSON.parse(this.uploadEditor.getValue()));
        this.uploadVisible = false;
      } catch (e) {
        this.$message.error(e.message);
        this.$refs.uploadJson.end();
      }
    },
    handleClear() {
      this.widgetForm = {
        list: [],
        config: {
          labelWidth: 100,
          labelPosition: "right",
          size: "small",
          customClass: ""
        }
      };

      this.widgetFormSelect = {};
    },
    getJSON() {
      return this.widgetForm;
    },
    setJSON(json) {
      this.widgetForm = json;

      if (json.list.length > 0) {
        this.widgetFormSelect = json.list[0];
      }
    },
    handleInput(val) {
      console.log(val);
      this.blank = val;
    },
    handleDataChange(field, value, data) {
      console.log(field, value, data);
    },
    handleCopyCode(flag) {
      let code = "";
      if (flag) {
        // 未预览时直接生成表单代码后复制
        code = genFormCode(this.widgetForm, this.widgetModels);
      } else {
        // 预览时复制已生成过的代码
        code = decodeURIComponent(this.$refs.codeRunRef.code);
      }
      const cpSuccess = copyText(code);
      if (cpSuccess) {
        this.$message.success(this.$t("fm.message.copySuccess"));
      }
    }
  },
  watch: {
    widgetForm: {
      deep: true,
      handler: function(val) {
        console.log(this.$refs.widgetForm);
      }
    },
    $lang: function(val) {
      this._loadComponents();
    }
  }
};
</script>

<style lang="scss">
.widget-empty {
  background-position: 50%;
}
.fm2-container.disabled {
  .el-form-item.active {
    outline: inherit !important;
    border: inherit !important;
  }
  .widget-view-drag {
    display: none !important;
  }
  .widget-view-action {
    display: none !important;
  }
}
</style>
