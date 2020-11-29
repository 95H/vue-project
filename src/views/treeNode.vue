<template>
  <div class="p-tree-node">
    <div
      :class="['p-tree-node-content', treeItem.editType&&'p-tree-node-edit', treeItem.checked&&'p-tree-node-content-checked']"
      :style="{paddingLeft: paddingLeft+'px'}"
    >
      <section class="p-tree-svg" @click="openChild">
        <ArrowTriangle
          :class="['p-tree-icon-svg', (treeItem.open)&&'p-tree-icon-rotate']"
          v-if="triangleShow"
        />
      </section>
      <div class="p-tree-node-check" @click="handleCheck(treeItem, $event)">
        <section class="p-tree-node-title">
          <article
            class="p-tree-node-name"
            @mouseenter="treeItemEnter"
            v-html="treeItem.name"
            v-show="treeItem.editType!=3"
          />
          <Input
            class="p-edit-input"
            v-show="editType==3"
            :maxLength="10"
            v-model="name"
            :autofocus="true"
            hideClear
            @blur="editName([...arguments, 'blur'])"
            placeholder="请输入"
          />
          <my-icon
            v-if="treeItem.editType==3"
            type="iconqueding"
            class="p-tree-node-more"
            @mousedown="editName([...arguments, 'click'])"
          ></my-icon>
          <my-icon
            v-else-if="!readOnly"
            type="icongengduo"
            class="p-tree-node-more"
            @click.stop="handleCheck(treeItem, $event, 'edit')"
          ></my-icon>
        </section>
      </div>
    </div>
    <div class="p-tree-child" v-if="triangleShow" v-show="treeItem.open">
      <TreeNode
        v-for="(item, ind) in treeItem.children"
        :key="item.id+'-'+ind"
        :treeItem="item"
        :triangleShow="!!(item.children&&item.children.length)"
        :index="`${index}-${ind}`"
        :readOnly="readOnly"
        :change="change"
      />
    </div>
  </div>
</template>

<script>
    import ArrowTriangle from "meri-design/lib/static/iconSvg/arrow_triangle.svg";
    import arrow from "meri-design/lib/static/iconSvg/arrow_right2.svg";
    export default {
        name: "TreeNode",
        components: {
            ArrowTriangle,
            arrow
        },
        props: {
            /**
             * 树形结构子项数据列表
             */
            treeItem: {
                type: Object,
                default: {},
            },
            /**
             * 点击某项
             */
            change: {
                type: Function,
                default: () => false,
            },
            /**
             * 下拉三角形是否显示
             */
            triangleShow: {
                type: Boolean,
                default: false,
            },
            /**
             * 索引
             */
            index: {
                type: String,
                default: "",
            },
            // 是否只读
            readOnly: {
                type: Boolean,
                default: false
            }
        },
        computed: {
            // 左边内边距
            paddingLeft() {
                return (this.index.split("-").length - 1) * 24 + 4;
            },
            editType() {
                set: {
                    this.name = this.treeItem.name;
                }
                get: {
                    return this.treeItem.editType;
                }
            },
        },
        data() {
            return {
                name: "",
                lockStatus: false, //当前编辑框的状态  true 为点击确认， false 为失焦
            };
        },
        mounted() {
            this.name = this.treeItem.name;
        },
        methods: {
            // 展开/收起
            openChild() {
                this.treeItem.open = !this.treeItem.open;
            },
            // 鼠标hover
            treeItemEnter(e) {
                const target = e.target;
                const {
                    clientWidth,
                    scrollWidth,
                    title
                } = target;
                if (!title && scrollWidth > clientWidth) target.title = target.innerText;
            },
            // 选择
            handleCheck(obj, e, type) {
                if (obj.editType == 3) return;
                this.change(obj, e, type);
                if (!type) {
                    const treeItem = this.treeItem;
                    treeItem.checked = true;
                } else {
                    this.treeItem.editType = true;
                }
            },
            // name操作
            async editName(params) {
                // 编辑保存
                if (params[1] == "click" && !this.lockStatus && this.name) {
                    this.lockStatus = true;
                    let name = this.name;
                    let addFlag = this.treeItem.addFlag;
                    let url =
                        addFlag == 1 || addFlag == 2 ?
                        this.$api.stock_addConsumableType :
                        this.$api.stock_updateConsumableType;
                    this.treeItem.addFlag = null;
                    this.treeItem.editType = null;
                    this.$store.commit("loading", {
                        spinning: true
                    });
                    // 调用后端方法
                    const {
                        data: {
                            Result,
                            ResultMsg
                        },
                    } = await this.$axios.post(url, {
                        code: this.treeItem.id,
                        parent: this.treeItem.parent ? this.treeItem.parent.id : null,
                        name,
                    });
                    this.$store.commit("loading", {
                        spinning: false
                    });
                    if (Result == "failure") {
                        return this.$message.error(ResultMsg);
                    }
                    this.$message.success("操作成功！");
                    this.treeItem.name = name;
                    this.treeItem.title = name;
                    // 重新刷新列表数据
                    this.$emit("change", {
                        id: this.treeItem.id
                    });
                }
                // 失焦且存在新增标志
                if (params[1] == "blur") {
                    if (!this.lockStatus && this.treeItem.addFlag) {
                        if (this.treeItem.parent) {
                            let index = this.treeItem.parent.children.findIndex(
                                (n) => n.addFlag
                            );
                            this.treeItem.parent.children.splice(index, 1);
                        } else {
                            // 因为是一级数据，可以通过$parent直接获取
                            let index = this.$parent.treeData.findIndex((n) => n.addFlag);
                            this.$parent.treeData.splice(index, 1);
                        }
                    }
                    this.treeItem.addFlag = null;
                    this.treeItem.editType = null;
                    this.lockStatus = false;
                }
            },
        },
    };
</script>