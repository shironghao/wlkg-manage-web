<template>
    <div>
      <v-card>
        <v-card-title>
          <v-btn @click="addBrand"  color="info">新增品牌</v-btn>
          <v-spacer/>
          <v-text-field single-line
                        hide-details
                        append-icon="search"
                        color="purple darken-2"
                        label="搜索"
                        v-model="search"
          ></v-text-field>
        </v-card-title>
        <v-divider/>
        <v-data-table
          :headers="headers"
          :items="brands"
          :pagination.sync="pagination"
          :total-items="totalBrands"
          :loading="loading"
          class="elevation-1"
        >
          <template slot="items" slot-scope="props">
            <td class="text-xs-center">{{ props.item.id }}</td>
            <td class="text-xs-center">{{ props.item.name }}</td>
            <td class="text-xs-center"><img v-if="props.item.image" width="130" height="40" :src="props.item.image"/><span v-else>无</span></td>
            <td class="text-xs-center">{{ props.item.letter }}</td>
            <td class="text-xs-center">
              <v-btn color="info" @click="editBrand(props.item)">编辑</v-btn>
              <v-btn color="yellow" @click="deleteBrand(props.item)">
                删除
              </v-btn>
            </td>
          </template>
        </v-data-table>

        <!--弹出的对话框-->

        <v-dialog v-model="show" max-width="600" scrollable >
          <v-card>
            <v-toolbar dark dense color="primary">
              <v-toolbar-title>{{isEdit ? '修改品牌' : '新增品牌'}}</v-toolbar-title>
              <v-spacer/>
              <v-btn icon @click="show = false">
                <v-icon>close</v-icon>
              </v-btn>
            </v-toolbar>
            <v-card-text class="px-5 py-2">
              <!-- 表单 -->
              <my-brand-form :oldBrand="oldBrand" :isEdit="isEdit" @close="show = false" :reload="getDataFromServer"/>
            </v-card-text>
          </v-card>
        </v-dialog>
<!--        <v-dialog max-width="500" v-model="show" persistent>-->
<!--          <v-card>-->
<!--            &lt;!&ndash;对话框的标题&ndash;&gt;-->
<!--            <v-toolbar dense dark color="primary">-->
<!--              <v-toolbar-title>{{isEdit ? '修改品牌' : '新增品牌'}}</v-toolbar-title>-->
<!--              <v-spacer/>-->
<!--              &lt;!&ndash;关闭窗口的按钮&ndash;&gt;-->
<!--              <v-btn icon @click="show=false"><v-icon>close</v-icon></v-btn>-->
<!--            </v-toolbar>-->
<!--            &lt;!&ndash;对话框的内容，表单&ndash;&gt;-->
<!--            <v-card-text class="px-5">-->
<!--              <my-brand-form @close="closeWindow" :oldBrand="oldBrand"/>-->
<!--&lt;!&ndash;              <my-brand-form @close="closeWindow"/>&ndash;&gt;-->
<!--            </v-card-text>-->
<!--          </v-card>-->
<!--        </v-dialog>-->

      </v-card>
    </div>
</template>

<script>
  // 导入自定义的表单组件
  import MyBrandForm from './MyBrandForm'

  export default {
    components:{
      MyBrandForm
    },
    data(){
          return {
            headers:[
              {
                text: 'Id',
                align: 'center',
                sortable: true,
                value: 'id'
              },
              { text: '名称', value: 'name', sortable: false, align: 'center'},
              { text: 'LOGO', value: 'image', sortable: false, align: 'center'},
              { text: '首字母', value: 'letter', sortable: false, align: 'center'},
              { text: '操作', sortable: false, align: 'center'},
            ],
            brands:[],
            pagination:{},
            totalBrands:0,
            loading:true,
            search:"",
            show:false,
            isEdit: false,
            oldBrand:{}
          }
        },
      methods:{
        closeWindow(){
          // 关闭窗口
          this.show = false;
          // 重新加载数据
          this.getDataFromServer();
        },
        getDataFromServer(){
          //this.brands ;
          //发送ajax请求到服务器端获取数据：axios,MVVM
          this.$http.get("/item/brand/page",
            {
              params:{
                key: this.search, // 搜索条件
                page: this.pagination.page,// 当前页
                rows: this.pagination.rowsPerPage,// 每页大小
                sortBy: this.pagination.sortBy,// 排序字段
                desc: this.pagination.descending// 是否降序
              }
            }
          ).then(resp=>{
            this.brands = resp.data.items;

            this.totalBrands = resp.data.total;
          })



          this.loading= false;

        },
        deleteBrand(item) {
          this.$message.confirm('此操作将永久删除该品牌, 是否继续?').then(() => {
            // 发起删除请求
            this.$http.delete("/item/brand/delete?id=" + item.id,)
              .then(() => {
                // 删除成功，重新加载数据
                this.$message.success("删除成功！");
                this.getDataFromServer();
              })
          }).catch(() => {
            this.$message.info("删除已取消！");
          });

        },
        addBrand(){
          this.isEdit=false;
          this.show=true;
          this.oldBrand=null;
        },
        editBrand(oldBrand){
          this.oldBrand = oldBrand;
          this.isEdit = true;
          this.show = true;
          // 查询商品分类信息，进行回显
          this.$http.get("/item/category/bid/" + oldBrand.id)
            .then(({data}) => {
                console.log(data)
                this.oldBrand.categories=data;
            })

        }
      },

      mounted() {
            this.getDataFromServer();
      },
      watch:{
          search(){
            this.getDataFromServer();
          },
          pagination(){
            this.getDataFromServer();
          }
      }
    }
</script>

<style scoped>

</style>
