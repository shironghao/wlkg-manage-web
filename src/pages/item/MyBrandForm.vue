<template>
    <div>
      <v-form v-model="valid" ref="myBrandForm">
        <v-text-field v-model="brand.name" :rules="nameRules" label="请输入品牌名称" required />
        <v-text-field v-model="brand.letter" mask="A" :rules="letterRules" label="请输入品牌首字母" required />
        <v-cascader
          url="/item/category/list"
          multiple
          required
          v-model="brand.categories"
          label="请选择商品分类"/>
        <v-layout row>
          <v-flex xs3>
            <span style="font-size: 16px; color: #444">品牌LOGO：</span>
          </v-flex>
          <v-flex>
            <v-upload
              v-model="brand.image"
              url="/upload/image"
              :multiple="false"
              :pic-width="250"
              :pic-height="90"
            />
          </v-flex>
        </v-layout>
        <v-layout class="my-4" row>
          <v-spacer/>
          <v-btn @click="submit" color="primary">提交</v-btn>
          <v-btn @click="clear" >重置</v-btn>
        </v-layout>

      </v-form>
    </div>
</template>

<script>
  export default {
    name: "my-brand-form",
    props:{
      oldBrand: Object,
      isEdit: {
        type:Boolean

      }
    },
    data() {
      return {
        valid:false, // 表单校验结果标记
        brand:{
          name:'', // 品牌名称
          letter:'', // 品牌首字母
          image:'',// 品牌logo
          categories:[], // 品牌所属的商品分类数组
        },
        nameRules:[
          v=>!!v||"品牌名称不能为空！",
          v=>v.length>1 ||"品牌名称至少2位"
        ],
        letterRules:[
          v=>!!v||"品牌首字母不能为空！",
          // v=>/^[A-Z]{1}$/.test(v) ||"品牌字母只能是A~Z的大写字母"
        ]
      }
    },
    watch: {
      oldBrand: {

        deep: true,
        handler(val) {
          console.log(val);
          // console.log(1);
          // console.log(this.brand);
          // console.log(2);
          // console.log(this.oldBrand);
          if(val){
            this.brand =  Object.deepCopy(val);

          }else{
            this.brand = {
              name: '',
              letter: '',
              image: '',
              categories: [],
            }
          }
        }
      }
    },
    methods:{
      submit(){
        // 提交表单
        //1、验证表单数据
        if(this.$refs.myBrandForm.validate()){
            // 2、定义一个请求参数对象，通过解构表达式来获取brand中的属性
            const {categories ,letter ,...params} = this.brand;
            // 3、数据库中只要保存分类的id即可，因此我们对categories的值进行处理,只保留id，并转为字符串
            params.cids = categories.map(c => c.id).join(",");

            // 4、将字母都处理为大写
            params.letter = letter.toUpperCase();
            console.log(params);


            // 5、将数据提交到后台
          //{name:zhangsan, age:20}->"name=zhangsan&age=20"
          this.$http({
            method: this.isEdit ? 'put' : 'post',
            url: '/item/brand',
            data: this.$qs.stringify(params)

          }).then(() => {
            // 关闭窗口
            this.$message.success("保存成功！");
            this.closeWindow();

          });
        }
      },
      clear(){
        // 重置表单
        this.$refs.myBrandForm.reset();
        // 需要手动清空商品分类
        this.categories = [];

      }
    }
  }

</script>

<style scoped>

</style>
