<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <tab-control class="tab-control" :titles="['流行','新款','精选']" @btnClick="btnClick" v-show = "isTabFixed" 
    ref="tabControl1"/>
    <scroll class="content" ref = "scroll" :probe-type = "3" 
    @scroll = "contentScroll" :pull-up-load = "true" @pullingUp = "loadMore">
      <home-swiper :banners="banners" @swiperImageLoad = "swiperImageLoad" />
      <recommend-view :recommends="recommends" />
      <featrue-view />
      <tab-control :titles="['流行','新款','精选']" @btnClick="btnClick" ref="tabControl"/>
      <goods-list :goods="goods[currentType].list" />
    </scroll>
    <back-top @click.native = "backClick" v-show = "isBackShow"/>
  </div>
</template>

<script>
import NavBar from "components/common/navbar/NavBar";
import HomeSwiper from "./childcomponents/HomeSwiper";
import RecommendView from "./childcomponents/RecommendView";
import FeatrueView from "./childcomponents/FeatureView";
import TabControl from "components/content/tabControl/TabControl";
import GoodsList from "components/content/goods/GoodsList";
import Scroll from "components/common/scroll/Scroll";
import BackTop from "components/content/backTop/BackTop";

import { getHomeMultidata, getHomeGoods } from "network/home";

export default {
  name: "Home",
  components: {
    NavBar,
    HomeSwiper,
    RecommendView,
    FeatrueView,
    TabControl,
    GoodsList,
    Scroll,
    BackTop
  },
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] }
      },
      currentType: "pop",
      isBackShow : false,
      isTabFixed: false,
      tabOffsetTop: 0
    };
  },
  created() {
    this.getHomeMultidata();
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  mounted(){
    const refresh = this.debounce(this.$refs.scroll.refresh)
    this.$bus.$on('imageLoad',() =>{
      refresh()
    })
    // console.log(this.$refs.tabControl.$el.offsetTop)
  },
  methods: {
    debounce(func,delay){
      let timer = null
      return function (...args){
        if(timer) clearTimeout (timer)
        timer = setTimeout(() => {
          func.apply(this,args)
        }, delay);
      }
    },
    swiperImageLoad(){
      this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop
      console.log(this.$refs.tabControl.$el)
    },
    btnClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
      }
      this.$refs.tabControl.currentIndex = index;
      this.$refs.tabControl1.currentIndex = index;
    },
    loadMore(){
      this.getHomeGoods(this.currentType)
      this.$refs.scroll.finishPullUp()

    },
    backClick(){
      this.$refs.scroll.scrollTo(0,0)
    },
    contentScroll(position){
      this.isBackShow = (-position.y) > 1000
      this.isTabFixed = (-position.y) > this.tabOffsetTop
    },
    getHomeMultidata() {
      getHomeMultidata().then(res => {
        // console.log(res);
        // this.result = res;
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then(res => {
        // console.log(res);
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;
      
      });
    }
  }
};
</script>

<style scoped>
#home {
  padding-top: 44px;
  height:100vh;
  position: relative;
}
.home-nav {
  background-color: var(--color-tint);
  color: #fff;
  text-align: center;

  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 9;
}
.tab-control {
  position: relative;
  /* top: 44px; */
  z-index: 9;
  background-color: #fff;
}
.content {
  /* height: 300px; */
  overflow: hidden;
  position:absolute;
  top:44px;
  bottom: 49px;
  left:0;
  right:0;
}
</style>
