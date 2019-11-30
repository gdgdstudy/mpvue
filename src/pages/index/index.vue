<template>
  <div class="container">
    <div class="city" bindtap="bindViewTap" v-if="basic">
      <div class="cityLeft">
        <image class="dwicon" src='../../static/images/curcity.png'></image>
        <text>当前城市：{{basic.city}}</text>
      </div>
      <div class="cityRight">
        <text class="update">最近更新时间：{{basic.update.loc}}</text>
        <image class="zbicon" src='../../static/images/update.png'></image>
      </div>
    </div>

    <div class="weather" bindtap="showcurid" v-if="now">
      <div class="aside">
        <text class="temperature">{{now.tmp}}℃</text>
        <text>{{now.cond.txt}} | {{now.wind.dir}}{{now.wind.sc}}级</text>
      </div>
      <div class="other">
        <div class="border_r"><text class="title">相对湿度</text><text class="info">{{now.hum}}%</text></div>
        <div class="border_r"><text class="title">降水量</text><text class="info">{{now.pcpn}}mm</text></div>
        <div><text class="title">能见度</text><text class="info">{{now.vis}}km</text></div>
      </div>
    </div>

    <div class="suggestion" v-if="suggestion">
      <text class="title">生活指数</text>
      <div class="list">
        <div class="list_l">
          <image src="../../static/images/icon/comf.png"></image>
          <text>舒适度指数</text>
        </div>
        <div class="list_r">
          <text class="list_t">{{suggestion.comf.brf}}</text>
          <text>{{suggestion.comf.txt}}</text>
        </div>
      </div>
      <div class="list">
        <div class="list_l">
          <image src="../../static/images/icon/cw.png"></image>
          <text>洗车指数</text>
        </div>
        <div class="list_r">
          <text class="list_t">{{suggestion.cw.brf}}</text>
          <text>{{suggestion.cw.txt}}</text>
        </div>
      </div>
      <div class="list">
        <div class="list_l">
          <image src="../../static/images/icon/drsg.png"></image>
          <text>穿衣指数</text>
        </div>
        <div class="list_r">
          <text class="list_t">{{suggestion.drsg.brf}}</text>
          <text>{{suggestion.drsg.txt}}</text>
        </div>
      </div>
      <div class="list">
        <div class="list_l">
          <image src="../../static/images/icon/flu.png"></image>
          <text>感冒指数</text>
        </div>
        <div class="list_r">
          <text class="list_t">{{suggestion.flu.brf}}</text>
          <text>{{suggestion.flu.txt}}</text>
        </div>
      </div>
      <div class="list">
        <div class="list_l">
          <image src="../../static/images/icon/sport.png"></image>
          <text>运动指数</text>
        </div>
        <div class="list_r">
          <text class="list_t">{{suggestion.sport.brf}}</text>
          <text>{{suggestion.sport.txt}}</text>
        </div>
      </div>
      <div class="list">
        <div class="list_l">
          <image src="../../static/images/icon/trav.png"></image>
          <text>旅游指数</text>
        </div>
        <div class="list_r">
          <text class="list_t">{{suggestion.trav.brf}}</text>
          <text>{{suggestion.trav.txt}}</text>
        </div>
      </div>
      <div class="list">
        <div class="list_l">
          <image src="../../static/images/icon/uv.png"></image>
          <text>紫外线指数</text>
        </div>
        <div class="list_r">
          <text class="list_t">{{suggestion.uv.brf}}</text>
          <text>{{suggestion.uv.txt}}</text>
        </div>
      </div>

    </div>
  </div>

</template>

<script>
// import card from '@/components/card'

export default {
  data() {
    return {
      basic: "",
      now: "",
      suggestion: "",
      app: "",
      curid: "CN101020100",
      forecast: null
    }
  },
  //自定义全局属性
  created() {
    this.app = getApp();
    this.app.curid = mpvue.getStorageSync('curid') || this.curid;
    this.setlocal('curid', this.app.curid);//调用全局方法
    this.onShow();
  },

  methods: {
    setlocal: function (id, val) {
      mpvue.setStorageSync(id, val);//API：设置本地缓存
    },
    onShow() {
      const that = this;
      mpvue.showToast({ title: '加载中', icon: 'loading', duration: 10000 });//设置加载模态框
      mpvue.request({//请求服务器，类似ajax
        // url: 'https://www.xiaoguge.cn/api/wxtest/now.php',
        url: 'https://free-api.heweather.com/v5/now',
        data: { city: this.app.curid, key: '01a7798b060b468abdad006ea3de4713' },//和风天气提供用户key，可自行注册获得
        header: { 'Content-Type': 'application/json' },
        success: function (res) {
          let d = res.data.HeWeather5[0];
          mpvue.hideToast();
          // d.now.cond.src = "http://files.heweather.com/cond_icon/" + d.now.cond.code + ".png";
          that.basic = d.basic;
          that.now = d.now;
          mpvue.request({
            url: 'https://free-api.heweather.com/v5/forecast',
           
           data: { city: that.app.curid, key: '01a7798b060b468abdad006ea3de4713' },//和风天气提供用户key，可自行注册获得
           // data: {
            //   "location": 116.4, 39.9,
            //   "key": '01a7798b060b468abdad006ea3de4713'
            // },
            header: { 'Content-Type': 'application/json' },
            success: function (res) {
              let result = res.data.HeWeather6[0];
              that.forecast = result;
            }
          });
        }
      });
      mpvue.request({
        url: 'https://www.xiaoguge.cn/api/wxtest/suggestion.php',
        data: { city: this.app.curid, key: '01a7798b060b468abdad006ea3de4713' },
        header: { 'Content-Type': 'application/json' },
        success: function (res) {
          let result = res.data.HeWeather5[0];
          that.suggestion = result.suggestion;
        }
      });
      


    },
    //4、页面事件绑定部分
    // bindViewTap() {
    //   // mpvue.switchTab({url: '../city/city'});
    // }//跳转菜单页面 

  }
}
</script>

<style scoped>
.city {
  width: 100%;
  padding: 3% 5%;
  background: #ddd;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: #333333;
  font-family: PingFangSC-Regular, PingFang SC;
}
.cityLeft,
.cityRight {
  display: flex;
  align-items: center;
}
.city .dwicon {
  width: 30rpx;
  height: 30rpx;
  float: left;
  margin: 5px;
}
.city text {
  font-size: 16px;
  color: #666;
  float: left;
}
.city .zbicon {
  width: 30rpx;
  height: 30rpx;
  float: right;
  margin: 8px 5px;
}
.city .update {
  font-size: 12px;
  float: right;
  width: 110px;
  text-align: center;
}

/*天气预报*/
.weather {
  width: 100%;
  overflow: hidden;
  background: #cef;
  color: #58b;
  padding: 5%;
}
.weather .section {
  float: left;
  width: 100rpx;
  height: 100rpx;
  display: block;
}
.weather .aside {
  float: right;
  width: 40%;
  margin-top: 5%;
  font-size: 12px;
}
.weather .aside .temperature {
  font-size: 36px;
  display: block;
}
.weather .other {
  clear: both;
  padding-top: 5%;
  display: flex;
}
.weather .other view {
  flex: 1;
  text-align: center;
}
.weather .other .border_r {
  border-right: solid 1px #bbd;
}
.weather .other .title {
  font-size: 12px;
  color: #bbd;
}
.weather .other .info {
  display: block;
  padding-top: 5%;
}

/*生活指数*/
.suggestion {
  margin-top: 5%;
}
.suggestion .title {
  display: block;
  padding: 3% 5%;
  background: #58b;
  color: #fff;
}
.suggestion .list {
  display: flex;
  font-size: 12px;
  margin-top: 1px;
}
.suggestion .list_l {
  flex: 1;
  text-align: center;
  background: #cef;
  color: #bbd;
  padding: 8px;
}
.suggestion .list_l image {
  width: 30rpx;
  height: 30rpx;
  display: block;
  margin: 0 auto;
}
.suggestion .list_r {
  flex: 4;
  background: #eee;
  padding: 8px;
  color: #aaa;
}
.suggestion .list_t {
  font-size: 14px;
  display: block;
  margin-bottom: 5px;
  color: #333;
}
</style>
