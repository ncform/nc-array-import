<template>
  <div>
    <ncform
      :form-schema="formSchema"
      form-name="your-form-name"
      v-model="formSchema.value"
      @submit="submit()"
    ></ncform>
    <hr />
    <pre>
      {{ JSON.stringify(formSchema.value, null, 2) }}
    </pre>
    <el-button @click="submit()">Submit</el-button>
  </div>
</template>

<script>
import ncArrayImport from "./components";

export default {
  created() {
    this.$ncformAddWidget({ name: "nc-array-import", widget: ncArrayImport });
  },
  data() {
    return {
      formSchema: {
        "type": "object",
        "properties": {
          "qq": {
            "type": "array",
            "items": {
              "type": "number",
              "ui": {},
            },
            "ui": {
              "columns": 6,
              "showLegend": false,
              "label": "QQ号",
              "widget": "nc-array-import",
              "widgetConfig": {
                "disableCollapse": true,
              }
            },
            "defaultValue": [ 1234567, 892310 ],
          },
          "tags": {
            "type": "array",
            "items": {
              "type": "string",
              "ui": {},
            },
            "ui": {
              "columns": 6,
              "showLegend": false,
              "label": "标签",
              "widget": "nc-array-import",
              "widgetConfig": {
                "disableCollapse": true,
              }
            }
          }
        },
        "value": {}
      }
    };
  },
  methods: {
    submit() {
      this.$ncformValidate("your-form-name").then(data => {
        if (data.result) {
          console.log(this.$data.formSchema.value);
        }
      });
    }
  }
};
</script>
