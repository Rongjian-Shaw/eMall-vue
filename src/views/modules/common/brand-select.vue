<template>
  <div>
    <el-select placeholder="请选择" v-model="brandId" filterable clearable>
      <el-option
        v-for="item in brands"
        :key="item.brandId"
        :label="item.brandName"
        :value="item.brandId"
      ></el-option>
    </el-select>
  </div>
</template>

<script>
/* eslint-disable */
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data () {
    // 这里存放数据
    return {
      catId: 0,
      brands: [
        {
          label: "a",
          value: 1
        }
      ],
      brandId: "",
      subscribe: null
    };
  },
  // 计算属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {
    brandId (val) {
      this.PubSub.publish("brandId", val);
    }
  },
  // 方法集合
  methods: {
    getCatBrands () {
      this.$http({
        url: this.$http.adornUrl("/product/categorybrandrelation/brands/list"),
        method: "get",
        params: this.$http.adornParams({
          catId: this.catId
        })
      }).then(({ data }) => {
        this.brands = data.data;
      });
    }
  },

  mounted () {
    // 监听三级分类消息的变化
    this.subscribe = PubSub.subscribe("catPath", (msg, val) => {
      this.catId = val[val.length - 1];
      this.getCatBrands();
    });
  },

  beforeDestroy () {
    PubSub.unsubscribe(this.subscribe); // 销毁订阅
  }

};
</script>
<style scoped>
</style>