<template>
  <el-form ref="data_model" :model="data_model" label-width="80px">
    <template v-for="item in data">
      <el-form-item
        v-if="item.type == 'input-str'"
        :label="item.label"
        :prop="item.key"
        :rules="item.required? rules._str:rules.str"
      >
        <el-input v-model="data_model[item.key]" :placeholder="item.placeholder"></el-input>
      </el-form-item>

      <el-form-item
        v-else-if="item.type == 'input-number'"
        :label="item.label"
        :prop="item.key"
        :rules="item.required? rules._number : rules.number"
      >
        <el-input v-model="data_model[item.key]" :placeholder="item.placeholder"></el-input>
      </el-form-item>

      <el-form-item
        v-else-if="item.type == 'input-switch'"
        :label="item.label"
        :prop="item.key"
        :rules="item.required? rules._boolean : rules.boolean"
      >
        <el-switch
          v-model="data_model[item.key]"
          :value="false"
          active-color="#13ce66"
          inactive-color="#ff4949"
        ></el-switch>
      </el-form-item>
    </template>
  </el-form>
</template>

<script>
import rules from "../../utils/roles.js";

export default {
  props: {
    //模板
    // formItems: [
    //     {
    //         label: '姓名',//*
    //         key: 'name',//*
    //         type: 'input-str',//*
    //         required: true, //是否必填
    //         placeholder: '0:String, 1:int, 2:Data, 3,statusOption, 4:timeLine' //占位符
    //     }
    // ],
    formItems: Array,
    formData: Object
  },
  data() {
    return {
      data_model: {},
      rules
    };
  },
  computed: {
    data() {
      console.log(this.formItems);
      
      return [...this.formItems];
    }
  },
  watch: {
    data_model: {
      handler(val) {
        console.log(val);
      },
      deep: true
    }
  },
  methods: {
    change() {
      this.$emit("change", [this.data_model]);
    },
    initData_model() {
      let self = this;
      if (this.formData) {
        this.data_model = {...this.formData};
        return;
      }
      this.data.forEach(item => {
        self.$set(this.data_model, item.key, "");
      });
    }
  },
  created() {
    this.initData_model();
  },
  mounted() {
    //   this.$refs['data_model'].resetFields();
    //   console.log(this.data_model);
  }
};
</script>

<style>
</style>
