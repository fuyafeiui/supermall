<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <tab-control class="tab-control" :titles="['流行','新款','精选']" @tabClick="tabClick" ref="tabControl1" v-show="isTabFixed"/>
    <scroll class="content" ref="scroll" :probe-type=3 @scroll="contentScroll" :pull-up-load=true @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends="recommends" />
      <feature-view />
      <tab-control :titles="['流行','新款','精选']" @tabClick="tabClick" ref="tabControl2" />
      <goods-list :goods="showGoods" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop" />
  </div>
</template>

<script>
  //1.导入的home组件
  import HomeSwiper from './childComps/HomeSwiper';
  import RecommendView from './childComps/RecommendView';
  import FeatureView from './childComps/FeatureView';

  //2.导入的项目公共组件
  import NavBar from "components/common/navbar/NavBar.vue";
  import TabControl from 'components/content/tabControl/TabControl';
  import GoodsList from 'components/content/goods/GoodsList';
  import Scroll from 'components/common/scroll/Scroll';
  import BackTop from 'components/content/backTop/BackTop'

  //3.导入的方法
  import { getHomeMultidata, getHomeGoods } from 'network/home.js';
  import { debounce } from "common/utils";
  export default {
    name: 'Home',
    components: {
      NavBar,
      HomeSwiper,
      RecommendView,
      FeatureView,
      TabControl,
      GoodsList,
      Scroll,
      BackTop
    },
    data() {
      return {
        result: null,
        banners: [],
        recommends: [],
        goods: {
          'pop': { page: 0, list: [] },
          'new': { page: 0, list: [] },
          'sell': { page: 0, list: [] }
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed: false,
        saveY: 0
      }
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list;
      }
    },
    created() {
      //1.请求多个数据
      this.getHomeMultidata()

      //2.请求商品信息
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')


    },
    destroyed () {
      console.log("destroyed")
    },
    deactivated () {
      this.saveY = this.$refs.scroll.getSaveY()
    },
    activated () {
      this.$refs.scroll.scrollTo(0,this.saveY,0);
      this.$refs.scroll.refresh(); 
    },
    mounted() {
      //1.监听图片加载完成时间监听
      const refresh = debounce(this.$refs.scroll.refresh, 50);
      this.$bus.$on("handleGoodsListImg", () => {
        refresh();
      });

      //2.获取吸顶的offsettop
      //所有的组件都有一个属性:$el 用于获取组件中的元素
    },
    methods: {
      /**
       * 事件监听的方法
       * 
      **/
      //1.切换商品列表类型
      tabClick(index) {
        switch (index) {
          case 0:
            this.currentType = 'pop'
            break
          case 1:
            this.currentType = 'new'
            break
          case 2:
            this.currentType = 'sell'
            break
        }
        this.$refs.tabControl1.currentIndex = index
        this.$refs.tabControl2.currentIndex = index
      },

      //2.返回顶部
      backClick() {
        this.$refs.scroll.scrollTo(0, 0)
      },

      //3.监听返回顶部图标位置，决定其显示或者隐藏的条件
      contentScroll(position) {
        //1.获取返回顶部图标显示的位置
        this.isShowBackTop = (-position.y) > 1000
        //2.获取吸顶的隐藏和显示
        this.isTabFixed = (-position.y) > this.tabOffsetTop
      },

      //4.上拉加载更多
      loadMore() {
        this.getHomeGoods(this.currentType)
      },
      swiperImageLoad() {
       this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
      },
      /*
      网络请求相关的方法
      **/
      //1.获取首页的数据
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          this.result = res;
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
        })
      },
      //2.获取商品列表数据(根据类型分类)
      getHomeGoods(type) {
        const page = this.goods[type].page + 1;
        getHomeGoods(type, page).then(res => {
          this.goods[type].list.push(...res.data.list);
          this.goods[type].page += 1;
          this.$refs.scroll.finishPullUp()
        });
      }
    }
  }
</script>

<style scoped>
  #home {
    /* padding-top: 44px; */
    height: 100vh;
    position: relative;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;
    /* position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9; */
  }

  /* 设置tabControl的滚动粘滞 */
  .tab-control {
    position: relative;
    z-index: 9;
  }

  .content {
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }

  /* 一种方法 */
  /* .content {
    height: calc(100% - 93px);
    overflow: hidden;
    margin-top: 44px;
  } */
</style>