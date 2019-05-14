<template>
    <div class="push-down-refresh">
        <div 
            class="refresh-text" 
            :style="'height:' + refreshTextHeight + ';lineHeight:' + refreshTextHeight + ';transitionDuration:' + transitionDuration + ';'"
        >
            <img v-if="refreshText == '加载中...'" class="xuanzhuan" src="./loding.png" alt="">
            <img v-else class="arrow" :style="endY - startY > 150 ? 'transform:rotate(180deg);' : ''" src="./arrow.png" alt="">
            {{refreshText}}
        </div>
        <div 
            :style="'top:' + refreshContentTop + ';transitionDuration:' + transitionDuration + ';'"
            class="refresh-content"
            @touchstart="touchstartR"
            @touchmove="touchmoveR"
            @touchend="touchendR"
        >
            <slot></slot>
        </div>
        <div 
            class="refresh-text-bottom refresh-text" 
            :style="'height:' + refreshTextBottomHeight + ';lineHeight:' + refreshTextBottomHeight + ';transitionDuration:' + transitionDuration + ';'"
        >
            <img v-if="refreshBottomText == '加载中...'" class="xuanzhuan" src="./loding.png" alt="">
            <img v-else class="arrow" :style="startY - endY > 150 ? '' : 'transform:rotate(180deg);'" src="./arrow.png" alt="">
            {{refreshBottomText}}
        </div>
    </div>
</template>

<script>
export default {
  name: 'XbcPushDownRefresh',
  props:{
      //用于监听的参数 当参数修改 结束下拉刷新 状态修改为加载完成
      stop:{
          type:Number,
          default:0
      },
      //是否开启 上拉加载更多 默认关闭
      pullUpRefreshMore:{
          type:Boolean,
          default:false
      },
      //是否开启刷新超时 刷新状态关闭 默认开启
      overtimeStopRefresh:{
          type:Boolean,
          default:true
      },
      //刷新 等待时间 毫秒 默认10000
      overtime:{
          type:Number,
          default:10000
      },
  },
  data () {
    return {
        //下拉刷新 提示文字
        refreshText: '下拉刷新',
        //上拉加载更多 提示文字
        refreshBottomText: '上拉加载更多',
        //提示文字行高
        refreshTextHeight:'0px',
        //提示文字行高
        refreshTextBottomHeight:'0px',
        //下拉刷新 内容距离顶部高度
        refreshContentTop:'0px',
        //下拉刷新 内容距离底部高度
        refreshContentBottom:'0px',
        //手指开始Y坐标
        startY:0,
        //手指结束Y坐标
        endY:0,
        //手指滑动中 ？ true为手指滑动中
        transitionDuration:'.0s',
        loading:false
    }
  },
  watch: {
    'stop'(){
        this.recover();
        if (this._timeout) {
            clearTimeout(this._timeout);
            this._timeout = null;
        }
    }
  },
  methods: {
      /**
       * 状态复原
       */
      recover(){
        this.startY = 0;
        this.endY = 0;
        this.loading = false;
        this.refreshTextHeight = '0px';
        this.refreshContentTop = '0px';
        this.refreshTextBottomHeight = '0px';
        this.refreshContentBottom = '0px';
        this.refreshText = '加载完成';
        this.refreshBottomText = '加载完成';
      },
    /**
     * 手指移动开始
     */
    touchstartR(e){
        if (this.loading) {
            return;
        }
        this.transitionDuration = '.0s';
        this.refreshText = '下拉刷新';
        this.refreshBottomText = '上拉加载更多';
        this.startY = e.touches[0].clientY;
    },
    /**
     * 手指移动中
     */
    touchmoveR(e){
        if (this.loading) {
            return;
        }
        let endY = e.touches[0].clientY;
        this.endY = endY;
        if (this.startY < endY) {
            this.refreshTextHeight = (endY - this.startY)*0.3 + 'px';
        }else if(this.pullUpRefreshMore){
            this.refreshContentTop = (endY - this.startY)*0.3 + 'px';
            this.refreshTextBottomHeight = (this.startY - endY)*0.3 + 'px';
        }
        if (endY - this.startY > 150) {
            this.refreshText = '释放刷新';
        }else if (endY - this.startY > 0 && endY - this.startY <= 150) {
            this.refreshText = '下拉刷新';
        }else if (this.startY - endY > 150 && this.pullUpRefreshMore) {
            this.refreshBottomText = '释放加载更多';
        }else if (this.startY - endY > 0 && this.startY - endY <= 150 && this.pullUpRefreshMore) {
            this.refreshBottomText = '上拉加载更多';
        }
    },
    /**
     * 滑动结束
     */
    touchendR(e){
        if (this.loading) {
            return;
        }
        this.transitionDuration = '.5s';
        let {startY,endY} = this;
        //大于此值 进行刷新逻辑
        if (endY - startY > 150) {
            //先 修改提示文字 回到50px
            this.loading = true;
            this.refreshText = '加载中...';
            this.refreshTextHeight = '50px';
            this.$emit('xbcPullDownRefresh');
            this.stopRefresh();
        }else if (endY - this.startY > 0 && endY - this.startY <= 150) { //否则直接返回原状态
            this.refreshTextHeight = '0px';
            this.refreshContentTop = '0px';
        }else if (this.startY - endY > 150 && this.pullUpRefreshMore) {
            //先 修改提示文字 回到50px
            this.loading = true;
            this.refreshBottomText = '加载中...';
            this.refreshTextBottomHeight = '50px';
            this.refreshContentTop = '-50px';
            this.$emit('xbcPullUpRefresh');
            this.stopRefresh();
        }else if (this.startY - endY > 0 && this.startY - endY <= 150 && this.pullUpRefreshMore) { 
            this.refreshTextBottomHeight = '0px';
            this.refreshContentTop = '0px';
        }
        this.startY = 0;
        this.endY = 0;
    },
    //结束 下拉刷新状态
    stopRefresh(){
        if (this.overtimeStopRefresh) {
            this._timeout = setTimeout(() => {
                this.recover();
            }, this.overtime);
        }
    }
  }
}
</script>

<style lang='stylus' scoped>
.push-down-refresh
    position relative
    height 100%
    overflow hidden
    .refresh-text
        text-align center
        transition-property line-height,height 
        overflow hidden
        background-color #fff
        outline 1px solid #ccc
        .arrow
            display inline-block
            transition transform .5s 
            transform rotate(0)
        .xuanzhuan
            animation xuanzhuan 1s linear infinite
    .transition-refresh-text-end
        transition: line-height .5s
    .refresh-content
        position relative
        height 100%
        overflow auto
        transition-property top
    .refresh-text-bottom
        position absolute
        bottom 0
        width 100%
@keyframes xuanzhuan {
    0% {
        transform rotate(0)
    }
    100%{
        transform rotate(360deg)
    }
}
</style>
