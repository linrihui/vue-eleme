<template>
  <div class="msite">
    <!--首页头部-->
    <header-top :title="address.name">
      <router-link class="header_search" slot="search" to="/search">
        <i class="iconfont icon-sousuo"></i>
      </router-link>
      <router-link class="header_login" slot="login" :to="userInfo._id?'/userinfo':'/login'">
        <span class="header_login_text">
          <span v-if="!userInfo._id">登录|注册</span>
          <i v-else class="iconfont icon-person"></i>
        </span>

      </router-link>
    </header-top>
    <!--首页导航-->
    <nav class="msite_nav">
      <div class="swiper-container" v-if="categories.length">
        <div class="swiper-wrapper">
          <div class="swiper-slide" v-for="categories in categoriesArr">
            <a href="javascript:" class="link_to_food" v-for="category in categories">
              <div class="food_container">
                <img :src="baseImageUrl+category.image_url">
              </div>
              <span>{{category.title}}</span>
            </a>
          </div>
        </div>
        <!-- Add Pagination -->
        <div class="swiper-pagination"></div>
      </div>
      <img src="./images/msite_back.svg" v-else>
    </nav>
    <!--首页附近商家-->
    <div class="msite_shop_list">
      <div class="shop_header">
        <i class="iconfont icon-xuanxiang"></i>
        <span class="shop_header_title">附近商家</span>
      </div>
      <shop-list/>
    </div>
  </div>
</template>

<script>
  import {mapState} from 'vuex'
  import Swiper from 'swiper'
  import 'swiper/dist/css/swiper.min.css'

  import HeaderTop from '../../components/HeaderTop/HeaderTop'
  import ShopList from '../../components/ShopList/ShopList'

  export default {
    data(){
      return{
        baseImageUrl:'https://fuss10.elemecdn.com'
      }
    },
    mounted () {
      this.$store.dispatch('getCategories')
      this.$store.dispatch('getShops')
    },
    computed:{
      ...mapState(['address','categories','userInfo']),
      /*
        根据categories一维数组生成二维数组
        小数组中元素个数最大是8
       */
      categoriesArr(){
        const {categories} = this
        const arr = []
        let minArr = []
        // 遍历categories
        categories.forEach(c => {
          // 如果minArr已经满了，创建一个新的
          if (minArr.length === 8){
            minArr = []
          }
          // 如果minArr是空的，将小数组保存到大数组中
          if(minArr.length === 0){
            arr.push(minArr)
          }
          // 将当前分类保存到小数组中
          minArr.push(c)
        })
        return this.categoriesArr = arr
      }
    },
    watch:{
      categories(value){  // categories数组中有数据了，在异步更新界面前执行
        // 界面更新就立即创建
        this.$nextTick(() => {  // 一旦完成界面更新 立即调用
          // 创建Swiper实例对象，来实现轮播
          new Swiper('.swiper-container',{
            loop: true, // 循环模式选项

            // 如果需要分页器
            pagination: {
              el: '.swiper-pagination',
            },

            // 如果需要滚动条
            scrollbar: {
              el: '.swiper-scrollbar',
            },
          })
        })
      }
    },
    components: {
      HeaderTop,
      ShopList
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixins.styl"
  .msite //首页
    width 100%

    .msite_nav
      bottom-border-1px(#e4e4e4)
      margin-top 45px
      height 200px
      background #fff

      .swiper-container
        width 100%
        height 100%

        .swiper-wrapper
          width 100%
          height 100%

          .swiper-slide
            display flex
            justify-content center
            align-items flex-start
            flex-wrap wrap

            .link_to_food
              width 25%

              .food_container
                display block
                width 100%
                text-align center
                padding-bottom 10px
                font-size 0

                img
                  display inline-block
                  width 50px
                  height 50px

              span
                display block
                width 100%
                text-align center
                font-size 13px
                color #666

        .swiper-pagination
          > span.swiper-pagination-bullet-active
            background #02a774

    .msite_shop_list
      top-border-1px(#e4e4e4)
      margin-top 10px
      background #fff

      .shop_header
        padding 10px 10px 0

        .shop_icon
          margin-left 5px
          color #999

        .shop_header_title
          color #999
          font-size 14px
          line-height 20px
</style>
