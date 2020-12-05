<template>
  <div class="__array-import-form-item">
    <legend v-if="schema.ui.legend && schema.ui.showLegend" @click="collapse()">
      {{_analyzeVal(schema.ui.legend)}}
      <i v-if="!mergeConfig.disableCollapse" class="el-collapse-item__arrow" :class="{'el-icon-arrow-up': !collapsed, 'el-icon-arrow-down': collapsed}"></i>
    </legend>

    <el-tabs v-model="currentTab" type="card" :editable="currentTab === 'preview'" @tab-remove="clearData">
      <div class="__array-import-form-item-split" v-show="!confirmed">
        <span>选择分割符号: </span>
        <el-select
          v-model="splitValue"
          placeholder="请选择"
          :size="mergeConfig.size"
        >
          <el-option label="制表符号" value="tab"></el-option>
          <el-option label="换行符" value="br"></el-option>
          <el-option label="逗号" value="comma"></el-option>
          <el-option label="其它" value="other"></el-option>
        </el-select>
        <el-input v-model="otherSplitValue" v-show="splitValue === 'other'" style="width: 100px;margin-left: 20px;"></el-input>
      </div>
      <el-tab-pane label="复制粘帖" name="paste" v-if="!confirmed" style="margin-top: 10px;">
        <el-input
          type="textarea"
          :rows="8"
          placeholder="请输入内容"
          v-model="pasteValue">
        </el-input>
        <el-button
          @click="confirm"
          v-show="!confirmed"
          style="margin-top: 10px;"
          :size="mergeConfig.size"
        >确认数据</el-button>
      </el-tab-pane>
      <el-tab-pane label="CSV 导入" name="import" v-if="!confirmed" style="margin-top: 10px;">
        <el-upload
          class="upload-demo"
          drag
          :http-request="confirmUpload"
          :multiple="false"
          :show-file-list="false"
          action="#"
        >
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">拖动 CSV 文件至此，或<em>点击上传</em></div>
        </el-upload>
      </el-tab-pane>
      <el-tab-pane label="预览数据" name="preview" v-if="confirmed">
        <div class="import_tip">
          <div class="success_tip">
            <span>成功导入数据</span><span class="success_cnt"> {{ schema.value.length }} </span>条

            <!--i
              title="复制成功数据"
              class="el-icon-document-copy"
              v-clipboard:copy="pasteValue"
              v-clipboard:success="onCopySuccess"
              v-clipboard:error="onCopyError"
            /-->
          </div>
          <div class="failed_tip">
            <span>导入失败数据</span><span class="failed_cnt"> {{ invalidValue.length }} </span>条
            <!--i
              title="复制失败数据"
              class="el-icon-document-copy"
              v-clipboard:copy="invalidValue.join('\n')"
              v-clipboard:success="onCopySuccess"
              v-clipboard:error="onCopyError"
            /-->
          </div>
          <div class="preview_text">
            <span>预览前 10 条数据: </span>
          </div>
        </div>
        <div class="preview_area">
          <ul class="preview_list">
            <li v-for="item in previewValue">{{ item }}</li>
          </ul>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<style lang="scss">
.__array-import-form-item {
  .import_tip {
    display: flex;
    .success_tip {
      .success_cnt {
        color: green;
      }
    }
    .failed_tip {
      margin-left: 20px;
      .failed_cnt {
        color: red;
      }
    }
    .preview_text {
      margin-left: 20px;
    }
    .el-icon-document-copy {
      cursor: pointer;
    }
  }
  .preview_area {
    padding-left: 10px;
    span {
      margin-left: -10px;
    }
  }
  .el-tabs__new-tab {
    display: none;
  }

  .preview_list {
    margin-left: 10px;
    margin: 0px;
    padding: 0px;
    background-color: #f7f8fa;
    padding: 16px;
    li {
      padding: 0px;
      height: 20px;
      line-height: 20px;
      font-size: 12px;
      list-style: none;
    }
  }
}
</style>

<script>
import ncformCommon from "@ncform/ncform-common";
const layoutArrayMixin = ncformCommon.mixins.vue.layoutArrayMixin;

const splitMap = {
  tab: '\t',
  br: '\n',
  comma: ',',
};

export default {
  mixins: [layoutArrayMixin],
  props: {
    value: {
      type: [Array]
    }
  },
  data() {
    return {
      currentTab: 'paste',
      splitValue: 'br',
      pasteValue: '',
      otherSplitValue: '',
      invalidValue: [],
      confirmed: false,
      previewValue: [],
    }
  },
  created() {
    const defaultValue = this.schema.defaultValue;
    if (defaultValue && defaultValue.length) {
      this.$set(this.schema, 'value', defaultValue);
      this.pasteValue = defaultValue.join(splitMap[this.splitValue]);
    } else {
      this.clearData();
    }
  },
  methods: {
    confirm() {
      const splitStr = splitMap[this.splitValue] || this.otherSplitValue;

      if (!splitStr) {
        return;
      }
      const value = [];
      const schemaType = (this.schema.items || {}).type;
      this.pasteValue.split(splitStr).forEach((item) => {
        let added = true;
        if (schemaType === 'string') {
          value.push(item);
        } else if (schemaType === 'number') {
          if (item === '') {
            added = false;
          } else {
            const val = Number(item);
            if (isNaN(val)) {
              added = false;
              this.invalidValue.push(item);
            } else {
              value.push(val);
            }
          }
        }
        if (added && this.previewValue.length < 10) {
          this.previewValue.push(value[value.length - 1]);
        }
      });
      this.$set(this.schema, 'value', value);
      this.confirmed = true;
      this.currentTab = 'preview';
    },
    confirmUpload(item) {
      item.file.text().then((text) => {
        this.pasteValue = text;
        this.confirm();
      });
    },
    clearData() {
      this.invalidValue = [];
      this.previewValue = [];
      this.$set(this.schema, 'value', []);
      this.confirmed = false;
      this.currentTab = 'paste';
    }
  }
};
</script>
