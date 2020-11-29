<template>
  <div class="p-tree">
    <TreeNode
      v-for="(item, ind) in treeData"
      :key="item.id+'-'+ind"
      :treeItem="item"
      :triangleShow="!!(item.children&&item.children.length)"
      :index="String(ind)"
      :change="change"
      :readOnly="readOnly"
    />
    <!-- 编辑选择框 -->
    <ul class="p-tree-edit-ul" v-show="showEdit" ref="editNode">
      <template v-for="el in editLi">
        <li :key="el.type" @click="editFun(el.type)" v-show="showEditLi(el.type)">{{el.name}}</li>
      </template>
</ul>
<!-- 编辑遮罩层 -->
<div v-show="showEdit" class="p-tree-edit-ul-wrap" @click="closeEdit"></div>
<!-- 删除对话框 -->
<Modal :show="modalStatusTip" title="弹窗提示" mode="tip" type="error" @close="modalClose">
    <template #content>{{deleteTile}}</template>
    <template #handle>
        <Button @click="modalClose" type="default">取消</Button>
        <Button @click="editFun(4, true)" type="error">删除</Button>
      </template>
</Modal>
</div>
</template>

<script>
    import TreeNode from "./treeNode";
    import {
        ChangeStatus
    } from "meri-design/lib/static/utils/TreeTool";
    export default {
        name: "Tree",
        components: {
            TreeNode
        },
        props: {
            /**
             * 树形结构数据列表
             */
            data: {
                type: Array,
                default: () => [],
            },
            // 是否只读
            readOnly: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                showEdit: false, //是否显示编辑框
                editLi: [{
                    name: "新增同级耗材类型",
                    type: 1
                }, {
                    name: "新增下级耗材类型",
                    type: 2
                }, {
                    name: "重命名",
                    type: 3
                }, {
                    name: "删除",
                    type: 4
                }, ],
                treeItem: {}, //当前选择的数据对象
                lastTreeItem: {}, // 上一次选择的数据对象
                lastEditItem: {}, // 上一次编辑的数据对象
                deleteTile: "", //删除操作时对话框的标题
                modalStatusTip: false, //是否显示删除对话框
            };
        },
        computed: {
            treeData: {
                get() {
                    return this.data;
                },
                set(newData) {
                    return newData;
                },
            },
        },
        methods: {
            /**
             * 点击某项
             * @param obj 返回当前点击对象
             * @param e 操作对象
             * @param type 操作类型
             */
            change(obj, e, type) {
                this.treeItem = obj;
                // 如果是编辑操作
                this.lastEditItem.editType = "";
                if (type === "edit") {
                    this.showEdit = true;
                    const dom = this.$refs.editNode;
                    // 判断显示的位置
                    this.$nextTick(() => {
                        const domHeight = dom.offsetHeight;
                        if (document.body.offsetHeight - e.pageY - domHeight > 0) {
                            dom.style.top = e.pageY + 12 + "px";
                        } else {
                            dom.style.top = e.pageY - domHeight - 12 + "px";
                        }
                        this.lastEditItem = obj;
                    });
                } else {
                    //  if (obj.id === this.lastTreeItem.id) return;
                    this.$emit("change", obj);
                    // 清除上一次的选中状态
                    this.lastTreeItem.checked = false;
                    // 将本次的选中数据存储起来
                    this.$nextTick(() => {
                        this.lastTreeItem = obj;
                    });
                }
            },
            // 获取node节点的level
            getParentLength(item) {
                let index = 0;
                if (item.parent) {
                    index = this.getParentLength(item.parent) + index + 1;
                }
                return index;
            },
            // 判断编辑显示的li(第四级不显示新增下级)
            showEditLi(type) {
                if (type == 2 && this.getParentLength(this.treeItem) == 3) {
                    return false;
                }
                return true;
            },
            /**
             * 编辑某项
             * @param type 操作类型
             */
            async editFun(type, next) {
                this.showEdit = false;
                const treeItem = this.treeItem;
                const parent = treeItem.parent || {};
                const children = parent.children || [];
                treeItem.editType = type;
                this.lastEditItem.editType = "";
                // 新增同级
                if (type == 1) {
                    treeItem.open = false;
                    if (children.length) {
                        let ind = children.findIndex((n) => n.id == treeItem.id);
                        children.splice(ind + 1, 0, {
                            name: "",
                            id: +new Date() + "",
                            checked: false,
                            addFlag: 1, //新增标识
                            editType: 3,
                            open: false,
                            parent: parent,
                        });
                    } else {
                        let findIndex = this.treeData.findIndex((n) => n.id == treeItem.id);
                        if (findIndex == this.treeData.length - 1) {
                            this.treeData.push({
                                name: "",
                                id: +new Date() + "",
                                checked: false,
                                addFlag: 1, //新增标识
                                editType: 3,
                                open: false,
                            });
                        } else {
                            this.treeData.splice(findIndex + 1, 0, {
                                name: "",
                                id: +new Date() + "",
                                checked: false,
                                addFlag: 1, //新增标识
                                editType: 3,
                                open: false,
                            });
                        }
                    }
                }
                // 新增下级
                else if (type == 2) {
                    treeItem.open = true;
                    if (treeItem.children && treeItem.children.length) {
                        treeItem.children.unshift({
                            name: "",
                            id: +new Date() + "",
                            checked: false,
                            addFlag: 2, //新增标识
                            editType: 3,
                            open: false,
                            parent: treeItem,
                        });
                    } else {
                        this.$set(treeItem, "children", [{
                            name: "",
                            id: +new Date() + "",
                            checked: false,
                            addFlag: 2, //新增标识
                            editType: 3,
                            open: false,
                            parent: treeItem,
                        }, ]);
                    }
                }
                // 重命名
                else if (type == 3) {
                    treeItem.editType = 3;
                }
                // 删除
                else if (type == 4) {
                    if (!next) {
                        this.modalStatusTip = true;
                        this.deleteTile = `确认删除${treeItem.name}${
            treeItem.children && treeItem.children.length
              ? "及全部下级分类"
              : ""
          }吗?`;
                        return;
                    }
                    // 调用删除方法
                    this.$store.commit("loading", {
                        spinning: true
                    });
                    this.modalStatusTip = false;
                    const {
                        data: {
                            Result,
                            ResultMsg
                        },
                    } = await this.$axios.post(this.$api.stock_deleteConsumableType, {
                        code: this.treeItem.id,
                    });
                    this.$store.commit("loading", {
                        spinning: false
                    });
                    if (Result == "failure") {
                        return this.$message.error(ResultMsg);
                    }
                    this.$message.success(ResultMsg);
                    if (parent && parent.children && parent.children.length) {
                        treeItem.parent.children.forEach((item, index) => {
                            if (item.id == treeItem.id) {
                                treeItem.parent.children.splice(index, 1);
                            }
                        });
                    } else {
                        let index = this.treeData.findIndex((n) => n.id == treeItem.id);
                        this.treeData.splice(index, 1);
                    }
                    // 判断删除的节点或者子节点是否是选中的节点，如果是选中的节点，需要回到选中全部
                    if (this.hasChecked(this.treeItem)) {
                        this.$emit("change", {
                            id: ""
                        });
                    }
                }
            },
            // 关闭删除对话框
            modalClose() {
                this.modalStatusTip = false;
            },
            // 判断是否有选中的节点
            hasChecked(item) {
                let checked = item.checked;
                if (checked) return true;
                if (item.children && item.children.length) {
                    item.children.forEach((n) => {
                        checked = this.hasChecked(n);
                    });
                }
                return checked;
            },
            // 关闭编辑框
            closeEdit() {
                this.showEdit = false;
                this.treeItem.editType = "";
            },
        },
    };
</script>


<style lang="less">
    .p-tree {
        width: 100%;
        overflow-x: hidden;
        font-size: 0;
        .p-input-icon-clear {
            display: none;
        }
    }
    
    .p-tree-node {
        width: 100%;
        height: 100%;
    }
    
    .p-tree-node-content {
        position: relative;
        width: 100%;
        display: flex;
        align-items: center;
        overflow: hidden;
        cursor: pointer;
        padding-top: 2px;
        padding-bottom: 2px;
        height: 40px;
        &:hover {
            background: rgba(239, 240, 241, 1);
            .p-tree-node-more {
                display: inline-block;
            }
        }
        &.p-tree-node-content-checked {
            background-color: #e1f2ff;
            .p-tree-node-check .p-tree-node-title .p-tree-node-name {
                color: #0091ff;
            }
        }
        .p-tree-svg {
            display: inline-block;
            vertical-align: middle;
            margin-right: 8px;
            width: 16px;
            height: 16px;
            line-height: 16px;
            text-align: center;
            .p-tree-icon-svg {
                vertical-align: text-bottom;
                transform: rotate(90deg);
                transition: transform 0.3s;
            }
            .p-tree-icon-rotate {
                transform: rotate(180deg);
            }
        }
        .p-tree-next {
            display: inline-block;
            vertical-align: middle;
            margin-right: 8px;
            width: 16px;
            height: 16px;
            line-height: 16px;
            text-align: center;
        }
        .p-tree-node-check {
            display: flex;
            align-items: center;
            overflow: hidden;
            flex: 1;
            .p-tree-node-title {
                display: flex;
                align-items: center;
                flex: 1;
                overflow: hidden;
                .p-tree-node-name {
                    flex: 1;
                    height: 38px;
                    line-height: 38px;
                    white-space: nowrap;
                    text-overflow: ellipsis;
                    overflow: hidden;
                    font-size: 14px;
                    color: #1f2429;
                }
            }
        }
        .p-tree-child {
            width: 100%;
        }
        .p-tree-node-more {
            display: none;
            margin-right: 12px;
            margin-left: 8px;
            svg {
                width: 16px;
                height: 16px;
            }
        }
        .p-tree-node-more-edit {
            display: inline-block;
        }
    }
    
    .p-tree-edit-ul {
        position: fixed;
        min-width: 160px;
        background: rgba(255, 255, 255, 1);
        box-shadow: 0px 2px 10px 0px rgba(31, 35, 41, 0.1);
        border-radius: 4px;
        border: 1px solid rgba(228, 229, 231, 1);
        left: 65px;
        z-index: 9999;
        li {
            font-size: 14px;
            line-height: 38px;
            color: rgba(31, 36, 41, 1);
            padding: 0 12px;
            cursor: pointer;
            &:hover {
                background: rgba(245, 246, 247, 1);
            }
        }
    }
    
    .p-tree-edit-ul-wrap {
        position: fixed;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
        z-index: 99;
    }
    
    .p-tree-node-edit {
        background: #eff0f1;
        .p-tree-node-more {
            display: inline-block;
        }
    }
</style>
