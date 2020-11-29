<template>
  <!-- 可编辑的树结构 -->
  <section class="s-tree-edlit">
    <!-- 搜索 -->
    <section class="s-tree-search">
      <my-icon type="iconsousuo"></my-icon>
      <input type="text" :placeholder="placeholder" v-model="search" />
    </section>
    <!-- 搜索展示 -->
    <ul v-show="search" class="s-tree-search-list">
      <template v-for="(item, index) in searchArr">
        <li
          :key="index"
          v-if="searchArr.length"
          v-html="handelName(item)"
          @mouseenter="treeItemEnter"
          @click="searchClick(item)"
        ></li>
      </template>
      <!-- 搜索无数据 -->
      <li class="no-data" v-if="searchArr.length==0">没有找到任何内容</li>
    </ul>
    <!-- 展示全部 -->
    <section class="s-tree-con" v-show="!search">
      <div :class="['all-select', !select&&'all-select-active']" @click="change({id:''})">全部</div>
      <!-- 有耗材 -->
      <section class="s-tree-con-wrap" v-if="dataArr.length">
        <tree-parent
          :data="dataArr"
          :readOnly="readOnly"
          ref="treeParent"
          @change="change"
          @option="option"
        />
      </section>
      <!-- 无耗材 -->
      <div class="p-tree-nodata" v-else>
        <article>暂无耗材类型</article>
        <Button @click="addTree" type="default" v-if="!readOnly">新建</Button>
      </div>
    </section>
  </section>
</template>

<script>
import treeParent from "./treeParent";
export default {
  components: {
    treeParent
  },
  props: {
    //输入框提示文字
    placeholder: {
      type: String,
      default: "请输入搜索内容..."
    },
    // tree数据
    data: {
      type: Array,
      default: () => []
    },
    // 是否只读
    readOnly: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      search: "", //搜索的数据
      reduceData: [], //一维数组数据
      select: "", // 当前选择的数据
      dataArr: [] //数据数组
    };
  },
  computed: {
    // 搜素结果
    searchArr() {
      if (!this.search || this.reduceData.length == 0) return [];
      return this.reduceData.filter(
        n => n.name && n.name.includes(this.search)
      );
    }
  },
  methods: {
    /**
     *  对树数据结构进行执行递归操作
     * @param list 需要遍历的数据
     * @param key 需要递归的key
     * @param cb 回调函数
     */
    handelTree(list = [], key, cb) {
      (function fn(list = [], parent) {
        list.forEach(item => {
          cb(item, parent);
          if (Array.isArray(item[key]) && item[key].length) fn(item[key], item);
        });
      })(list);
    },
    /**
     *  处理搜索情况下的name
     * @param item 数据对象
     */
    handelName(item, level) {
      let name = "";
      if (item.parent) {
        name = this.handelName(item.parent, true) + "/" + name;
      }
      let searchName = item.name.replace(
        this.search,
        `<span style="color: #0091ff;font-size: 14px;">${this.search}</span>`
      );
      return name ? name + searchName : searchName;
    },
    // 搜索列表鼠标hover
    treeItemEnter(e) {
      const target = e.target;
      const { clientWidth, scrollWidth, title } = target;
      if (!title && scrollWidth > clientWidth) target.title = target.innerText;
    },
    // 点击搜索列
    searchClick(item) {
      this.search = "";
      item.checked = true;
      const treeParentDom = this.$refs.treeParent;
      treeParentDom.change(item);
      // 需要判断选择的元素是否在视野内，如果不在则需要滚动定位
      try {
        let serchDom = treeParentDom.$el.querySelector(
          ".p-tree .p-tree-node-content-checked"
        );
        let searchH =
          serchDom.offsetTop +
          serchDom.offsetHeight -
          document.body.offsetHeight;
        this.$nextTick(() => {
          treeParentDom.$el.scrollTop = 500;
        });
      } catch {}
    },
    // 数据改变
    change(obj, e, type) {
      this.select = obj.id;
      if (!this.select) {
        this.$refs.treeParent.treeItem.checked = false;
        this.$refs.treeParent.lastTreeItem.checked = false;
      }
      this.$emit("change", obj.id);
    },
    // 新增
    addTree() {
      this.dataArr.push({
        name: "",
        id: +new Date() + "",
        checked: false,
        addFlag: true, //新增标识
        editType: 3,
        open: false
      });
    },
    // 增删改查操作
    option(type) {
      this.$emit(type, type);
    }
  },
  watch: {
    data(n) {
      this.dataArr = this._.cloneDeep(n);
      this.handelTree(this.dataArr, "children", (item, parent) => {
        this.$set(item, "parent", parent);
        this.$set(item, "editType", "");
        this.$set(item, "checked", false);
        this.$set(item, "open", true);
        this.reduceData.push(item);
        if (item.checked) {
          this.$refs.treeParent.treeItem = item;
          this.$refs.treeParent.lastTreeItem = item;
        }
      });
    }
  }
};
</script>

<style lang="less" scoped>
.s-tree-edlit {
  display: flex;
  flex-direction: column;
  width: 100%;
  background-color: inherit;
  overflow: hidden;
  height: 100%;
  .s-tree-search {
    line-height: 22px;
    padding: 16px;
    border-bottom: 1px solid #e4e6e7;
    display: flex;
    align-items: center;
    input {
      flex: 1;
      border: none;
      outline: none;
      margin-left: 8px;
      &::placeholder {
        color: #9ca1a9;
      }
    }
  }
  .s-tree-con {
    flex: 1;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    position: relative;
    .all-select {
      color: #1f2429;
      padding: 11px 0;
      padding-left: 16px;
      line-height: 19px;
      border-bottom: 1px solid #e4e6e7;
      cursor: pointer;
    }
    .all-select-active {
      color: #0091ff;
      background-color: #e1f2ff;
    }
    .s-tree-con-wrap {
      flex: 1;
      overflow-y: auto;
      padding: 7px 0;
      box-sizing: border-box;
      &:hover {
        overflow-y: overlay;
      }
      .p-tree{
        padding:0 12px;
        box-sizing: border-box;
      }
    }
    .p-tree-nodata {
      flex: 1;
      position: absolute;
      top: 40%;
      left: 50%;
      transform: translate(-50%, -50%);
      article {
        color: #646c73;
        margin-bottom: 16px;
      }
    }
  }
  .s-tree-search-list {
    overflow-x: hidden;
    overflow-y: scroll;
    li {
      line-height: 22px;
      padding: 9px 16px;
      cursor: pointer;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      &:hover {
        background: #eff0f1;
      }
    }
    .no-data {
      color: rgba(141, 147, 153, 1);
      margin-top: 104px;
      text-align: center;
    }
  }
}
</style>