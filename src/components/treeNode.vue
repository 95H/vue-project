<template>
    <div class='p-tree-node'>
        <div :class='["p-tree-node-content", treeItem.checked&&"p-tree-node-content-checked"]' :style='{paddingLeft: paddingLeft+"px"}'>
            <section class='p-tree-svg' @click='openChild'>
                <div :class='["p-tree-icon-svg", (treeItem.open)&&"p-tree-icon-rotate"]' v-if='triangleShow'>
                    <Icon name='revoke' />
                </div>
            </section>s
            <div class='p-tree-node-check' @click='handleCheck(treeItem, $event)'>
                <section class='p-tree-node-title'>
                    <article class='p-tree-node-name' @mouseenter='treeItemEnter' v-html='treeItem.name' />
                    <div class='p-tree-node-more' v-if='!treeItem.children' @click.stop='handleCheck(treeItem, $event,"edit")'>
                        <Icon name='more' hoverColor='#0091ff' width='20' height='20' />
                    </div>
                </section>
            </div>
        </div>
        <div class='p-tree-child' v-if='triangleShow' v-show='treeItem.open'>
            <TreeNode
                v-for='(item, ind) in treeItem.children'
                :key='item.id+"-"+ind'
                :treeItem='item'
                :triangleShow='!!(item.children&&item.children.length)'
                :index='`${index}-${ind}`'
                :change='change'
            />
        </div>
    </div>
</template>

<script>
import { Icon } from 'meri-design'

export default {
    name: 'TreeNode',
    components: {
        Icon,
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
            default: '',
        },
    },
    computed: {
        // 左边内边距
        paddingLeft() {
            return (this.index.split('-').length - 1) * 24 + 4
        },
    },
    data() {
        return {
            name: '',
            lockStatus: false, //当前编辑框的状态  true 为点击确认， false 为失焦
        }
    },
    mounted() {
        this.name = this.treeItem.name
    },
    methods: {
        // 展开/收起
        openChild() {
            this.treeItem.open = !this.treeItem.open
        },
        // 鼠标hover
        treeItemEnter(e) {
            const target = e.target
            const { clientWidth, scrollWidth, title } = target
            if (!title && scrollWidth > clientWidth) target.title = target.innerText
        },
        // 选择
        handleCheck(obj, e, type) {
            this.change(obj, e, type)
            if (!type) {
                const treeItem = this.treeItem
                treeItem.checked = true
            } else {
                this.treeItem.editType = true
            }
        },
    },
}
</script>
<style lang="less" scoped>
.p-tree-node {
    width: 100%;
    height: 100%;
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
                    text-align: left;
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
        }
    }
}
</style>