<template>
    <div class='p-tree'>
        <TreeNode
            v-for='(item, ind) in treeData'
            :key='item.id+"-"+ind'
            :treeItem='item'
            :triangleShow='!!(item.children&&item.children.length)'
            :index='String(ind)'
            :change='change'
        />
        <!-- 编辑选择框 -->
        <ul class='p-tree-edit-ul' v-show='showEdit' v-clickoutside='handleClose' ref='editNode'>
            <template v-for='el in editLi'>
                <li :key='el.type' @click='editFun(el.type)'>{{el.name}}</li>
            </template>
        </ul>
        <!-- 删除对话框 -->
        <Modal :show='modalStatusTip' title='确认要删除此巡更点吗？' mode='tip' type='error' @close='modalClose'>
            <template #handle>
                <Button @click='modalClose' type='default'>取消</Button>
                <Button @click='editFun(4, true)' type='error'>删除</Button>
            </template>
        </Modal>
    </div>
</template>

<script>
import TreeNode from './treeNode'
import { Modal } from 'meri-design'
const clickoutside = {
    // 初始化指令
    bind(el, binding, vnode) {
        function documentHandler(e) {
            // 这里判断点击的元素是否是本身，是本身，则返回
            if (el.contains(e.target)) {
                return false
            }
            // 判断指令中是否绑定了函数
            if (binding.expression) {
                // 如果绑定了函数 则调用那个函数，此处binding.value就是handleClose方法
                binding.value(e)
            }
        }
        // 给当前元素绑定个私有变量，方便在unbind中可以解除事件监听
        el.__vueClickOutside__ = documentHandler
        document.addEventListener('click', documentHandler)
    },
    unbind(el, binding) {
        // 解除事件监听
        document.removeEventListener('click', el.__vueClickOutside__)
        delete el.__vueClickOutside__
    },
}
export default {
    name: 'Tree',
    components: {
        TreeNode,
        Modal,
    },
    props: {
        /**
         * 树形结构数据列表
         */
        data: {
            type: Array,
            default: () => [],
        },
    },
    data() {
        return {
            showEdit: false, //是否显示编辑框
            editLi: [
                {
                    name: '下载二维码',
                    type: 1,
                },
                {
                    name: '编辑巡更点',
                    type: 2,
                },
                {
                    name: '删除巡更点',
                    type: 4,
                },
            ],
            treeItem: {}, //当前选择的数据对象
            lastTreeItem: {}, // 上一次选择的数据对象
            lastEditItem: {}, // 上一次编辑的数据对象
            modalStatusTip: false, //是否显示删除对话框
        }
    },
    computed: {
        treeData: {
            get() {
                return this.data
            },
            set(newData) {
                return newData
            },
        },
    },
    directives: { clickoutside },
    mounted(){
        this.defaultData(this.treeData)
    },
    methods: {
        // 查询默认值
        defaultData(arr){
            let _this = this
            let item = null
            try{
                arr.forEach((val,key)=>{
                    if(val.children && val.children.length> 0){
                        _this.defaultData(val.children)
                        throw '终止'
                    }else{
                        val.checked = true
                        _this.change(val)
                        throw '终止'
                    }
                })
            }catch(e){
                // console.log(e)
            }
        },
        handleClose(e) {
            this.showEdit = false
        },
        /**
         * 点击某项
         * @param obj 返回当前点击对象
         * @param e 操作对象
         * @param type 操作类型
         */
        change(obj, e, type) {
            this.treeItem = obj
            // 如果是编辑操作
            // this.lastEditItem.editType = ''
            if (type === 'edit') {
                this.showEdit = true
                const dom = this.$refs.editNode
                // 判断显示的位置
                this.$nextTick(() => {
                    const domHeight = dom.offsetHeight
                    if (document.body.offsetHeight - e.pageY - domHeight > 0) {
                        dom.style.top = e.pageY + 12 + 'px'
                    } else {
                        dom.style.top = e.pageY - domHeight - 12 + 'px'
                    }
                    this.lastEditItem = obj
                })
            } else {
                //  if (obj.id === this.lastTreeItem.id) return;
                this.$emit('change', obj)
                // 清除上一次的选中状态
                this.lastTreeItem.checked = false
                // 将本次的选中数据存储起来
                this.$nextTick(() => {
                    this.lastTreeItem = obj
                })
            }
        },
        // 获取node节点的level
        getParentLength(item) {
            let index = 0
            if (item.parent) {
                index = this.getParentLength(item.parent) + index + 1
            }
            return index
        },
        /**
         * 编辑某项
         * @param type 操作类型
         */
        async editFun(type, next) {
            this.showEdit = false
            const treeItem = this.treeItem
            const parent = treeItem.parent || {}
            const children = parent.children || []
            // 删除
            // else
            if (type == 4) {
                if (!next) {
                    this.modalStatusTip = true
                    return
                }
                // 调用删除方法
                this.$store.commit('loading', {
                    spinning: true,
                })
                this.modalStatusTip = false
                const {
                    data: { Result, ResultMsg },
                } = await this.$axios.post(this.$api.stock_deleteConsumableType, {
                    code: this.treeItem.id,
                })
                this.$store.commit('loading', {
                    spinning: false,
                })
                if (Result == 'failure') {
                    return this.$message.error(ResultMsg)
                }
                this.$message.success(ResultMsg)
                if (parent && parent.children && parent.children.length) {
                    treeItem.parent.children.forEach((item, index) => {
                        if (item.id == treeItem.id) {
                            treeItem.parent.children.splice(index, 1)
                        }
                    })
                } else {
                    let index = this.treeData.findIndex((n) => n.id == treeItem.id)
                    this.treeData.splice(index, 1)
                }
                // 判断删除的节点或者子节点是否是选中的节点，如果是选中的节点，需要回到选中全部
                if (this.hasChecked(this.treeItem)) {
                    this.$emit('change', {
                        id: '',
                    })
                }
            }
        },
        // 关闭删除对话框
        modalClose() {
            this.modalStatusTip = false
        },
        // 判断是否有选中的节点
        hasChecked(item) {
            let checked = item.checked
            if (checked) return true
            if (item.children && item.children.length) {
                item.children.forEach((n) => {
                    checked = this.hasChecked(n)
                })
            }
            return checked
        },
    },
}
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
.p-tree-edit-ul {
    position: fixed;
    width: 120px;
    background: rgba(255, 255, 255, 1);
    box-shadow: 0px 2px 10px 0px rgba(31, 35, 41, 0.1);
    border-radius: 4px;
    border: 1px solid #e4e5e7;
    left: 154px;
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
</style>
