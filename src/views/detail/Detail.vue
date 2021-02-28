<template>
  <div>
    <detail-nav-bar />
    <detail-swiper :topImages="topImages"/>
    <detail-base-info :goods="goods"/>
  </div>
</template>

<script>
import DetailNavBar from './childComps/DetailNavBar';
import DetailSwiper from './childComps/DetailSwiper';
import DetailBaseInfo from './childComps/DetailBaseInfo';
//导入的方法
import {getDetailInfo,Goods} from 'network/detail.js'
export default {
  name: 'Detail',
  data() {
    return {
      iid: null,
      topImages: [],
      goods: {}
    }
  },
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo
  },  
  created () {
    this.iid = this.$route.params.iid;
    this.getDetailInfo(this.iid)
  },
  methods: {
    getDetailInfo(iid) {
      getDetailInfo(iid).then(res => {

        console.log(res.result)
        const data = res.result
        //1.获取头部的图片轮播数据
        this.topImages = data.itemInfo.topImages
        //2.获取商品信息
        this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)
        console.log(this.goods)

      })
    }
  }
}
</script>

<style scoped>

</style>