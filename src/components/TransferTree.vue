<template>
  <div class="transfer-tree-wrap">
    <div class="tree-select">
      <div class="tree-select__title">
        <el-checkbox
          @change="clickAllSelect"
          :class="{
            'is-indeterminate': indeterminate,
          }"
          v-model="leftCheck"
        ></el-checkbox>
        <div class="title">备选项</div>
      </div>
      <div class="tree-select__tree">
        <el-tree
          ref="leftTree"
          :data="leftData"
          :props="defaultProps"
          show-checkbox
          node-key="id"
          @check="handleLeftTreeClick"
        />
      </div>
    </div>

    <div class="transfer—btn">
      <el-button
        type="primary"
        icon="el-icon-arrow-right"
        @click="transferRight"
      ></el-button>
      <el-button
        type="primary"
        icon="el-icon-arrow-left"
        @click="transferLeft"
      ></el-button>
    </div>

    <div class="tree-select">
      <div class="tree-select__title">
        <el-checkbox
          @change="clickAllSelect"
          :class="{
            'is-indeterminate': indeterminate,
          }"
          v-model="leftCheck"
        ></el-checkbox>
        <div class="title">备选项</div>
      </div>
      <div class="tree-select__tree">
        <el-tree
          ref="rightTree"
          :data="rightData"
          :props="defaultProps"
          show-checkbox
          node-key="id"
          @check="handleLeftTreeClick"
        />
      </div>
    </div>
  </div>
</template>

<script>
const mockTreeData = [
  {
    id: "1",
    label: "1",
    children: [
      {
        id: "1-1",
        label: "1-1",
        pid: "1",
      },
      {
        id: "1-2",
        label: "1-2",
        pid: "1",
      },
      {
        id: "1-3",
        label: "1-3",
        pid: "1",
      },
    ],
  },
  {
    id: "2",
    label: "2",
    children: [
      {
        id: "2-1",
        label: "2-1",
        pid: "2",
      },
      {
        id: "2-2",
        label: "2-2",
        pid: "2",
      },
      {
        id: "2-3",
        label: "2-3",
        pid: "2",
      },
    ],
  },
  {
    id: "3",
    label: "3",
    children: [
      {
        id: "3-1",
        label: "3-1",
        pid: "3",
      },
      {
        id: "3-2",
        label: "3-2",
        pid: "3",
      },
      {
        id: "3-3",
        label: "3-3",
        pid: "3",
      },
    ],
  },
];
export default {
  name: "transfer-tree",
  props: {
    data: {
      type: Array,
      default: () => mockTreeData,
    },
  },
  data() {
    return {
      // leftCheck: false,
      defaultProps: {
        children: "children",
        label: "label",
      },
      leftData: [],
      rightData: [],
      leftSelKeys: [],
      rightSelKeys: [],
    };
  },
  mounted() {
    this.leftData = this.data;
    this.rightData = [];
  },
  computed: {
    indeterminate() {
      return (
        this.leftSelKeys.length > 0 &&
        this.leftSelKeys.filter((item) => item.children).length !=
          this.leftData.length
      );
    },
    leftCheck() {
      const flag =
        this.leftSelKeys.filter((item) => item.children).length ==
          this.leftData.length && this.leftSelKeys.length != 0;
      return flag;
    },
  },
  methods: {
    // 点击向右穿梭
    transferRight() {
      // (leafOnly, includeHalfChecked) 接收两个 boolean 类型的参数，
      // 1. 是否只是叶子节点，默认值为 false 2. 是否包含半选节点，默认值为 false
      const checkedNodes = this.$refs.leftTree.getCheckedNodes(false, true); // 包含半选
      const checkedKeys = this.$refs.leftTree.getCheckedKeys(false);

      const copyNodes = JSON.parse(JSON.stringify(checkedNodes));
      copyNodes.forEach((x) => {
        x.children = [];
        if (!this.$refs.rightTree.getNode(x.id)) {
          this.$refs.rightTree.append(x, x.pid);
        }
      });

      checkedKeys.forEach((x) => {
        this.$refs.leftTree.remove(x);
      });
      this.afterToward();
    },
    // 点击向左穿梭
    transferLeft() {
      const checkedNodes = this.$refs.rightTree.getCheckedNodes(false, true); // 包含半选
      const checkedKeys = this.$refs.rightTree.getCheckedKeys(false);

      const copyNodes = JSON.parse(JSON.stringify(checkedNodes));
      copyNodes.forEach((x) => {
        x.children = [];
        if (!this.$refs.leftTree.getNode(x.id)) {
          this.$refs.leftTree.append(x, x.pid);
        }
      });

      checkedKeys.forEach((x) => {
        this.$refs.rightTree.remove(x);
      });

      this.afterToward();
    },
    /**左侧树点击 */
    handleLeftTreeClick(data, sel) {
      this.leftSelKeys = sel.checkedNodes;
    },
    /**左侧全选 */
    clickAllSelect(v) {
      if (!v) return this.$refs.leftTree.setCheckedNodes([]);
      this.$refs.leftTree.setCheckedNodes(this.leftData);
      // this.towardsRight();
    },
    /**右侧全选 */
    clickCancelAllSelect() {
      this.$refs.rightTree.setCheckedNodes(this.rightData);
      // this.towardsLeft();
    },
    // 数据穿梭后
    afterToward() {
      this.$refs.leftTree.setCheckedKeys([]);
      this.$refs.rightTree.setCheckedKeys([]);
      console.log(this.leftData, this.rightData);
      this.$emit("getdata", {
        leftData: this.leftData,
        rightData: this.rightData,
      });
    },
    // 数据回显
    renderTree() {
      this.leftData = [mockTreeData[1]];
      this.rightData = [mockTreeData[0], mockTreeData[2]];
    },
  },
};
</script>

<style lang="scss">
.transfer-tree-wrap {
  display: flex;
  justify-content: space-around;
  .tree-select {
    border: 1px solid #ccc;
    height: 270px;
    width: 200px;
    color: #fff;
    position: relative;
    .tree-select__title {
      background: #a0cfff;
      height: 30px;
      line-height: 30px;
      /* text-align: center; */
      border-bottom: 1px solid #ccc;
      cursor: pointer;
      display: flex;
      padding: 0 12px;
      .title {
        margin-left: 8px;
      }
    }
  }
  .transfer—btn {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
}
.el-checkbox {
  &.is-indeterminate {
    .el-checkbox__inner {
      background: #409EFF;
      border-color: #409EFF;
      transform: all 0.3s;
      &::before {
        content: "";
        position: absolute;
        display: block;
        background-color: #FFF;
        height: 2px;
        -webkit-transform: scale(0.5);
        transform: scale(0.5);
        left: 0;
        right: 0;
        top: 5px;
      }
    }
  }
}
</style>