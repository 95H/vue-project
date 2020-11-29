<template>
    <div class='tree-container'>
        <tree></tree>
    </div>
</template>

<script>
import tree from './tree'
export default {
    name: 'HelloWorld',
    data() {
        return {}
    },
    components: {
        tree,
    },
}
</script>
<style scoped lang='less'>
.tree-container {
    width: 280px;
    height: 100vh;
    border: 1px solid #f7f9fa;
}
</style>
