<template>
  <div class="recommend" ref="recommend">
		<scroll ref="scroll" class="recommend-content" :data="discList">
      <div>
  			<div v-if="recommends.length" class="slider-wrapper">
  			   <slider>
              <div v-for="item in recommends">
                <a :href="item.linkUrl">
                  <img class="needsclick" @load="loadImage" :src="item.picUrl">
                </a>
              </div>   
           </slider>
  			</div>
  			<div class="recommend-list">
  				<h1 class="list-title">热门歌单推荐</h1>
  				<ul>
  				  <li class="item" v-for="item in discList" @click="selectItem(item)">
              <div class="icon">
                <img width="60" height="60" v-lazy="item.imgurl">
              </div>    
              <div class="text">
                <h2 class="name" v-html="item.creator.name"></h2>
                <p class="desc" v-html="item.dissname"></p>
              </div>
            </li>
  				</ul>
  			</div>
      </div>
      <div class="loading-container" v-show="!discList.length">
        <loading></loading>
      </div>
		</scroll>
    <router-view></router-view>
  </div>
</template>

<script>
  import Loading from 'base/loading/loading'
  import Scroll from 'base/scroll/scroll'
  import Slider from 'base/slider/slider'
	import {getRecommend, getDiscList} from 'api/recommend'
	import {ERR_OK} from 'api/config'
  import {playlistMixin} from 'common/js/mixin'
  import {mapMutations} from 'vuex'
	export default {
    mixins: [playlistMixin],
    data() {
      return {
        recommends: [],
        discList: [],
        checkLoaded: false
      }
    },
		created() {
      this._getRecommend()
      this._getDiscList()
		},
		methods: {
      // 歌单点击
      selectItem(item) {
        this.$router.push({
          path: `/recommend/${item.dissid}`
        })
        this.setDisc(item)
      },
      // 设置bottom
      handlePlaylist(playlist) {
        const bottom = playlist.length > 0 ? '60px' : ''
        this.$refs.recommend.style.bottom = bottom
        this.$refs.scroll.refresh()
      },
      // 获取轮播图数据
			_getRecommend() {
				getRecommend().then((res) => {
					if (res.code === ERR_OK) {
						this.recommends = res.data.slider
					}
				})
			},
      // 获取歌单数据
      _getDiscList() {
        getDiscList().then((res) => {
          if (res.code === ERR_OK) {
            this.discList = res.data.list
          }
        })
      },
      // 图片加载完成
      loadImage() {
        if (!this.checkLoaded) {
          this.$refs.scroll.refresh()
          this.checkLoaded = true
        }
      },
      ...mapMutations({
        setDisc: 'SET_DISC'
      })
		},
    components: {
      Slider,
      Scroll,
      Loading
    }
	}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .recommend
    position: fixed
    width: 100%
    top: 88px
    bottom: 0
    .recommend-content
      height: 100%
      overflow: hidden
      .slider-wrapper
        position: relative
        width: 100%
        overflow: hidden
      .recommend-list
        .list-title
          height: 65px
          line-height: 65px
          text-align: center
          font-size: $font-size-medium
          color: $color-theme
        .item
          display: flex
          box-sizing: border-box
          align-items: center
          padding: 0 20px 20px 20px
          .icon
            flex: 0 0 60px
            width: 60px
            padding-right: 20px
          .text
            display: flex
            flex-direction: column
            justify-content: center
            flex: 1
            line-height: 20px
            overflow: hidden
            font-size: $font-size-medium
            .name
              margin-bottom: 10px
              color: $color-text
            .desc
              color: $color-text-d
      .loading-container
        position: absolute
        width: 100%
        top: 50%
        transform: translateY(-50%)
</style>