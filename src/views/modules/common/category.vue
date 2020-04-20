<template>
    <el-tree
            :data="menus"
            :props="defaultProps"
            node-key="catId"
            ref="menuTree"
            @node-click="clickNodeTree"
    >

    </el-tree>
</template>

<script>
    export default {
        data() {
            return {
                menus: [],
                defaultProps: {
                    children: "children",
                    label: "name"
                }
            };
        },
        methods: {
            getDataList() {
                this.dataListLoading = true;
                this.$http({
                    url: this.$http.adornUrl("/product/category/list/tree"),
                    method: "get"
                }).then(({data}) => {
                    this.menus = data.data;
                });
            },
            clickNodeTree(data,node,components) {
                this.$emit("tree-node-click",data,node,components);
            }
        },
        created() {
            this.getDataList();
        }
    }
</script>

<style>

</style>