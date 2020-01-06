<template>
  <v-card>
      <v-flex xs12 sm10>
        <v-tree url="/item/category/list"
                :isEdit="isEdit"
                @handleAdd="handleAdd"
                @handleEdit="handleEdit"
                @handleDelete="handleDelete"
                @handleClick="handleClick"
        />
      </v-flex>
  </v-card>
</template>

<script>
  export default {
    name: "category",
    data() {
      return {
        isEdit:true,
      }
    },
    methods: {
      handleAdd(node) {
        console.log(node),
        this.$http.post("/item/category/add",{


            isParent: node.isParent === true ? 1 : 0,

            name: "新的节点",
            parentId: node.parentId,

            sort: node.sort



        }).then(data=>{
          this.$message.success("添加成功！");
        })



        // console.log("add .... ");
        // console.log(node);
      },
      handleEdit(id, name) {
        this.$http.put("/item/category/edit",{
          id:id,
          name :name

        }).then(data=>{
          this.$message.success("修改成功！");
        })
       // console.log("edit... id: " + id + ", name: " + name)
      },
      handleDelete(id) {
        console.log(id);
        this.$http.delete("/item/category/delete",{
          params:{
            id:id
          }

        }).then(data=>{
          this.$message.success("删除成功")
        })
        console.log("delete ... " + id)
      },
      handleClick(node) {
        console.log(node)
      }
    }
  };
</script>

<style scoped>

</style>
