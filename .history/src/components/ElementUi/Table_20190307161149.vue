<template>
  <el-table
    :data="tableData"
    style="width: 100%"
    height="500"
    :span-method="arraySpanMethod"
    @selection-change="handleSelectionChange"
    stripe
    border
    @header-dragend="handleHeaderDragend"
  >
    <el-table-column type="selection" width="55"></el-table-column>
    <el-table-column v-if="isHideNumber" type="index" width="50" fixed></el-table-column>
    <template v-for="title  in tableTitle">
      <!--特殊字段 -->
      <el-table-column v-if="title.topType==='createDate'" :label="title.headName" width="180">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span>{{ scope.row.createDate | date-format}}</span>
        </template>
      </el-table-column>

      <el-table-column
        width="150"
        v-else-if="title.topType==='userExpirationDate'"
        sortable
        :label="title.headName"
      >
        <template slot-scope="scope">
          <span
            v-if="scope.row.userExpirationDate!==0"
          >{{ scope.row.userExpirationDate | date-format}}</span>
          <span v-if="scope.row.userExpirationDate===0">始终有效</span>
        </template>
      </el-table-column>
      <el-table-column
        width="150"
        v-else-if="title.topType==='pwdValidityPeriod'"
        sortable
        :label="title.headName"
      >
        <template slot-scope="scope">
          <span
            v-if="scope.row.pwdValidityPeriod!==0"
          >{{ scope.row.pwdValidityPeriod | date-format}}</span>
          <span v-if="scope.row.pwdValidityPeriod===0">始终有效</span>
        </template>
      </el-table-column>

      <el-table-column
        v-else-if="title.topType==='landingTime'"
        :label="title.headName"
        width="180"
      >
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span>{{ scope.row.landingTime | date-format}}</span>
        </template>
      </el-table-column>

      <el-table-column v-else-if="title.topType==='eDate'" :label="title.headName" width="180">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span>{{ scope.row.userExpirationDate | date-format}}</span>
        </template>
      </el-table-column>

      <el-table-column v-else-if="title.topType==='modifyDate'" :label="title.headName" width="180">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span>{{ scope.row.modifyDate | date-format}}</span>
        </template>
      </el-table-column>

      <el-table-column v-else-if="title.topType==='createDate'" :label="title.headName" width="180">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span>{{ scope.row.createDate | date-format}}</span>
        </template>
      </el-table-column>

      <el-table-column v-else-if="title.topType==='auditDate'" :label="title.headName" width="180">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span>{{ scope.row.createDate | date-format}}</span>
        </template>
      </el-table-column>

      <el-table-column
        v-else-if="title.topType==='accountStatus'"
        :label="title.headName"
        width="180"
        :formatter="account"
      ></el-table-column>

      <el-table-column
        v-else-if="title.topType==='statusOptions'"
        :label="title.headName"
        width="180"
        :formatter="statusOptions"
      ></el-table-column>

      <el-table-column
        v-else
        sortable
        :fixed="!!title.isFixed"
        :label="title.headName"
        :prop="title.topType"
        :show-overflow-tooltip="true"
      ></el-table-column>
    </template>
  </el-table>
</template>
<script>
export default {
  data() {
    return {};
  },
  props: {
    tableData: Array,
    tableTitle: Array
  },
  methods: {
    //是否隐藏编号
    isHideNumber() {
      let flag = false;
      this.tableTitle.forEach(item => {
        if (item.headName == "编号") {
          flag = true;
        }
      });
      return flag;
    },
    //tabale表头上下箭头 排序
    arraySpanMethod({ row, column, rowIndex, columnIndex }) {
      // if (rowIndex % 2 === 0) {
      //   if (columnIndex === 0) {
      //     return [1, 2]
      //   } else if (columnIndex === 1) {
      //     return [0, 0]
      //   }
      // }
    },
    //点击选项 checkbox 按钮 获得val赋值给 传给页面
    handleSelectionChange(val) {
      this.$emit("checkboxValue", val);
    },
    //当拖动表头改变了列的宽度的时候会触发该事件
    handleHeaderDragend(newWidth, oldWidth, column, event) {
      //内边距左右各10  每个字符宽度按 15的宽度计算
      column.minWidth = column.label.length * 15 + 20;
      //如果有排序的图标则加24  排序的箭头宽度 26
      if (column.sortable) {
        column.minWidth += 24;
      }
      column.width =
        column.width < column.minWidth ? column.minWidth : column.width;
    },
    //table 账号状态 转换显示
    account: function(row) {
      return row.accountStatus === 0
        ? "正常"
        : row.accountStatus === 1
        ? "冻结"
        : row.accountStatus === 2
        ? "禁用"
        : "";
    },
    statusOptions: function(row) {
      if (row.statusOptions.length) {
        console.log(row);
        return row.statusOptions
          .map(item => {
            return item.name;
          })
          .join(",");
      } else {
        return "";
      }
    }
  }
};
</script>

<style scoped lang="scss">
.el-tooltip__popper {
  max-width: 500px;
  line-height: 180%;
}
</style>
