<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" ref="nav" @titleCilck="titleCilck"></detail-nav-bar>
    <Scroll class="content" ref="scroll" @scroll="contentScroll" :probe-type="3">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods"></detail-base-info>
      <detail-shop-info :shop="shop"></detail-shop-info>
      <detail-goods-info :detail-info="detailInfo" @loadImgEvent="imageLoad"></detail-goods-info>
      <detail-param-info ref="params" :param-info="paramInfo"></detail-param-info>
      <detail-comment-info ref="comment" :comment-info="commentInfo"></detail-comment-info>
      <goods-list ref="recommend" :goods="recommends"></goods-list>
    </Scroll>
    <detail-bottom-bar @addCart="addToCart"></detail-bottom-bar>
      <back-top @click.native="backClick"
     v-show="isShowBackTop" 
    ></back-top>
  </div>
</template>

<script>
 import DetailNavBar from './childComps/DetailNavBar'
 import DetailSwiper from './childComps/DetailSwiper'
 import DetailBaseInfo from './childComps/DetailBaseInfo'
 import DetailShopInfo from './childComps/DetailShopInfo'
 import DetailGoodsInfo from './childComps/DetailGoodsInfo'
 import DetailParamInfo from './childComps/DetailParamInfo'
 import DetailCommentInfo from './childComps/DetailCommentInfo'
 import DetailBottomBar from './childComps/DetailBottomBar'





import {debounce} from 'common/utils/util'
import {itemListenerMixin,backTopMixin} from 'common/mixin'

 import Scroll from 'components/common/scroll/Scroll'
 import GoodsList from 'components/content/goods/GoodsList'
 import {mapActions} from 'vuex'


 import {getDetail,getRecommend,Goods,Shop,GoodsParam} from 'network/detail'

  export default {
    name:'Detail',
    components:{
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommentInfo,
      DetailBottomBar,
      GoodsList,
      Scroll
    },
    mixins:[itemListenerMixin,backTopMixin],
    data() {
      return {
        iid:null,
        topImages:[],
        goods:{},
        shop:{},
        detailInfo:{},
        paramInfo:{},
        commentInfo:{},
        recommends:[],
        themeTopYs:[],
        getThemeTopY:null,
        currentIndex:0,

      }
    },
    created() {
      // 1.保存传入的iid
      this.iid = this.$route.params.iid

      // 2.根据iid请求s详情数据
      getDetail(this.iid).then(res => {
          // console.log(res);
          // 获取顶部图片数据
          const data = res.result;
          this.topImages = data.itemInfo.topImages;

          // 获取商品信息
          this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)
          // console.log(this.goods);
          // 创建店铺信息
          this.shop = new Shop(data.shopInfo)

          // 保存商品的详情数据
          this.detailInfo = data.detailInfo;

          // 获取参数的信息
          this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)
       
          // 取出评论信息
          if (data.rate.cRate !== 0) {
            this.commentInfo = data.rate.list[0]
          }

          // this.$nextTick(() => {
          //   // 根据最新的数据，对应的DOM是已经被渲染出来
          //   // 但是图片依然没有加载完(目前获取到的offsetTop不包含图片的高度)
          //   this.themeTopYs = []
          //   this.themeTopYs.push(0)
          //   this.themeTopYs.push(this.$refs.params.$el.offsetTop)
          //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
          //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)

            // console.log(this.themeTopYs);
          // })
         

       },
        // 请求推荐数据
        getRecommend().then(res => {
          // console.log(res);
          this.recommends = res.data.list
        }),
        // 2.使用防抖
        this.getThemeTopY = debounce(() => {
          this.themeTopYs = []
          this.themeTopYs.push(0)
          this.themeTopYs.push(this.$refs.params.$el.offsetTop -44)
          this.themeTopYs.push(this.$refs.comment.$el.offsetTop -44)
          this.themeTopYs.push(this.$refs.recommend.$el.offsetTop -44)
          this.themeTopYs.push(Number.MAX_VALUE)


          // console.log(this.themeTopYs);
        },100)
      )},
      methods: {
        ...mapActions(['addCart']),
        imageLoad() {
          this.$refs.scroll.refresh()

          this.getThemeTopY()
          // 1.直接使用  2.防抖
          // this.themeTopYs = []
          // this.themeTopYs.push(0)
          // this.themeTopYs.push(this.$refs.params.$el.offsetTop -44)
          // this.themeTopYs.push(this.$refs.comment.$el.offsetTop -44)
          // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop -44)

          // console.log(this.themeTopYs);
        },
        titleCilck(index) {
          // console.log(index);
          this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],200)
        },
        contentScroll(position) {
          // 获取y值
          const positionY = -position.y

          // positionY主题中值进行对比
          let length = this.themeTopYs.length
          for (let i = 0; i < length-1; i++) {
            // if ((this.currentIndex !== i) && ((i < length - 1 && positionY >= this.themeTopYs[i] && 
            // positionY < this.themeTopYs[i+1]) || (i === length - 1 && positionY >= this.themeTopYs[i]))) {
            //   this.currentIndex = i;
            //   // console.log(this.currentIndex);
            //   this.$refs.nav.currentIndex = this.currentIndex
            // }
            if ((this.currentIndex !== i) && (positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])) {
              this.currentIndex = i;
              this.$refs.nav.currentIndex = this.currentIndex
            }

          }
          // 是否显示回到顶部
          this.isShowBackTop = (-position.y) > 1000

        },
        addToCart() {
          //获取购物车需要展示的信息
          const product = {}
          product.image = this.topImages[0]
          product.title = this.goods.title
          product.desc = this.goods.desc
          product.newPrice = this.goods.realPrice
          product.iid = this.iid

          // 将商品添加到购物车(1.Promise 2.mapActions)
          // this.$store.commit('addCart',product)

          this.addCart(product).then(res => {
            
            this.$toast.show(res,2000)
          })

          // this.$store.dispatch('addCart',product).then(res => {
          //   console.log(res);
          // })
        }
      
      },
      // mounted() {
      //    // 图片加载完成的时间监听
      //   const refresh = debounce(this.$refs.scroll.refresh)
      //   //  监听item中图片加载完成
      //   this.$bus.$on('loadImgEvent',() => {
      //     // console.log('----');
      //     refresh()
      //   })
      // },
      destroyed() {
        this.$bus.$off('itemImgLoad',this.itemListenerMixin)
      },
  }
</script>

<style lang="less" scoped>
#detail {
  position: relative;
  z-index: 9;
  background-color: #fff;
}
.detail-nav{
  position: relative;
  z-index: 9;
  background-color: #fff;
}
.content {
  height: calc(100vh - 44px - 49px);
}
</style>