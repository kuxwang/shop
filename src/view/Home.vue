<template>
  <div class="content">
    <div class="top-logo">
      <img :src="shops.logo" alt= "">
      <!--麦麦国际-->
    </div>
    <div class="top-bar">
      <div class="logo"><img :src="avatar"/></div>
      <div class="input" @click="goSearch()">
        <span class="iconfont">&#xe651;</span>
        {{searchtext}}
      </div>
      <div class="share"  @click="share()">
        <span class="iconfont">&#xe71d;</span>
      </div>
    </div>
    <div class="scroll-view">
      <mt-swipe class="banner" :auto="2000" :show-indicators="true">
        <mt-swipe-item v-for="(i, x) in slider" :key="x">
          <img class="silder" :src="i.thumb" @click="specil(i.app_link)">
        </mt-swipe-item>
      </mt-swipe>
      <v-colrow :list="hot" :title="shops.alias.ishot" v-if="hashot"></v-colrow>
      <v-colrow :list="istime" :title="shops.alias.istime" v-if="hastime"></v-colrow>
      <v-colcol :list="newgoods" :title="allproduct" v-if="hasnew"></v-colcol>
      <div class="bottom-img">
        <!--<img class="bottom-pic" :src="logo2"/>-->
        <img class="bottom-pic" :src="adv"/>
      </div>
    </div>
    <v-tabbar></v-tabbar>
    <router-view></router-view>
  </div>
</template>
<script>
  import vTabbar from '../components/mode/Tabbar';
  import vSearch from '../components/mode/search';
  import vColrow from '../components/common/columnRow';
  import vColcol from '../components/common/columnCol';
  import defalutAvatar from '../assets/images/defaultlogo.png'
  import {fn} from '../config/myUtils';
  import { Advs, memberInfo,Attributes,Share,Adv,iGetSessionKey,Shops } from '../api/api';
  import {mapMutations, mapGetters, mapState} from 'vuex'
  import {MessageBox} from 'mint-ui';
  import {_webapp} from './../config/hook.js'

  export default{
    data () {
      return {
        img1: require('../assets/images/confirmorder-01.jpg'),
        logo: require('../assets/images/logo.png'),
        logo2: require('../assets/images/bottom.png'),
        slider:[],
        selected: 1,
        isSearch:false,
        avatar: defalutAvatar,
        newgoods:[],
        hot:[],
        istime:[],
        hashot:false,
        hasnew:false,
        hastime:false,
        adv:'',
        islogin:false,
        sharedata:'',
        shops:{},
        headerlogo:'',
        searchtext:'',
        allproduct:''
      }
    },
    methods: {
      getSilder(){
        let params = {
          data: {}
        };
        Advs(params, (res) => {
          if (res.statusCode === 1) {
            this.slider = res.data;
            console.log(this.slider)
          }
        })
      },
      toggle(){
        this.isSearch=!this.isSearch;
      },
      getUserInfo () {
        let _this=this;
        memberInfo({data: {}}, res => {
          if(res.statusCode == 1){
            console.log('首页用户信息')
            console.log(res.data)
            _this.avatar = res.data.parent_avatar ||defalutAvatar
            console.log('上级头像')
            console.log(res.data.parent_avatar)
            console.log(_this.avatar)
            _this.islogin = true;
            _this.sharedata = res.data.share;
//            console.log('用户已经登录')
//            console.log(_this.sharedata)
          }else{

          }
        })
      },
      getNew(){
        let parmas = {
          data: {
            attributes: "isnew:1",
            page: 1,
            psize: 10,
            fields:'description,title,id,productprice,marketprice,thumb,app_thumb_url,total'
          }
        }
        Attributes(parmas, (res) => {
          console.log('new')
          console.log(res)
          if (res.statusCode === 1) {
            this.newgoods = res.data;
            if(res.data.length){
              this.hasnew=true
            }
          }else {
            this.hasnew=false
          }
        })
      },
      getHot(){
        let parmas = {
          data: {
            attributes: "ishot:1",
            page: 1,
            psize: 10,
            fields:'description,title,id,productprice,marketprice,thumb,app_thumb_url,total'
          }
        }
        Attributes(parmas, (res) => {
          console.log(res);
          if (res.statusCode === 1) {
            this.hot = res.data;
            if(res.data.length){
              this.hashot=true
            }
          }else {
            this.hashot=false
          }
        })
      },
      getTime(){
        let parmas = {
          data: {
            attributes: "istime:1",
            page: 1,
            psize: 10,
            fields:'description,title,id,productprice,marketprice,thumb,app_thumb_url，total'
          }
        }
        Attributes(parmas, (res) => {
          console.log(res);
          if (res.statusCode === 1) {
            this.istime = res.data;
            if(res.data.length){
              this.hastime=true
            }
          }else {
            this.hastime=false
          }
        })
      },
      goSearch(){
        this.$router.push('search')
      },
      share(){
        let _this=this;
        let params=_this.sharedata;
        console.log('分享参数')
        console.log(params)
        _webapp.checkLogin(function (res) {
          if(res.statusCode==1){
            memberInfo({data: {}}, res => {
              if (res.statusCode == 1) {
              /*  console.log('首页用户信息')
                console.log(res.data)*/
                _this.sharedata = res.data.share;
                console.log('用户已经登录,分享')
                console.log(_this.sharedata)
                Share( res.data.share,(res) => {
                  console.log(1)
                })
              } else {
//                console.log(用户接口请求错误)
              }

            })



          }else {
            MessageBox({title: '很抱歉，您还未登陆', message: '是否去登陆', showCancelButton: true,confirmButtonText:'去登陆'}).then(action => {
              if (action === 'confirm') {//表示点击了确定
                _webapp.nativeLogin();
                console.log('去登陆')
              } else if (action === 'cancel') {//表示点击了取消
                console.log('不去登陆')
              }
            })
          }
        })
      },
      getAdv(){
        let params = {
          data: {
            'identification': 'index'
          }
        };
        Adv(params, (res) => {
          if (res.statusCode === 1) {
            this.adv = res.data.thumb;
          }
        })
      },
      getShops(){
        let params={
          data:{}
        }
        Shops(params,res=>{
          if(res.statusCode===1){
            console.log('shops数据')
            console.log(res.data)
            this.shops=res.data;
            this.searchtext=res.data.other.searchtext;
            this.allproduct=res.data.other.allproduct;
//            this.headerlogo=res.data.logo;
//            this.shops.map(v,i,a)
            /*console.log(Object.keys(this.shops))
            for(let i in this.shops){
              console.log(i)
            }*/
          }
        })
      },
      specil(v){
        this.$router.push({name:'special',query:{key:v}})
      }
    },
    mounted () {
      let _this=this;
      this.getSilder();
      this.getNew();
      this.getHot();
      this.getTime();
      this.getAdv();
      this.getShops()
      _webapp.checkLogin(function (res) {
        if(res.statusCode==1){
          console.log('JS BRIDEG')
          _this.getUserInfo();
        }
      })
//      this.getUserInfo();
    },
    components: {
      vTabbar,
      vSearch,
      vColrow,
      vColcol
    },
    computed: {
      ...mapGetters([
        'haslogo'
      ]),
    }

  }
</script>
<style lang="less" scoped>
  @import '../assets/css/reset/reset.css';
  @import '../assets/css/reset/common.less';
  @import '../assets/css/fonts/iconfont.css';

  .content {
    .page-view();
    .top-logo {
      height: auto;
      background: #fff;
      color: #fff;
      font-size: .20rem;
      font-weight: bold;
      /*line-height: auto;*/
      img {
        width: 100%;
        height: 100%;
        display: block;
      }
    }
    font-family: PingFang !important;
  }

  .top-bar {
    display: flex;
    /*height: .45rem;*/
    height: .35rem;
    background-color: #fff;
    font-family: PingFang !important;

    .logo {
      width: .4rem;
      font-size: .21rem;
      color: #000000;
      padding: .02rem 0.05rem .05rem .1rem;

      img {
        /*width: 100%;*/
        /*height: 100%;*/
        width:.3rem;
        height:.3rem;
        border-radius: .15rem;
      }
    }
    .share {
      position: relative;
      /*width: .51rem;*/
      width: .45rem;
      left: -0.15rem;
      top: -.05rem;
      .iconfont {
        position: absolute;
        z-index: 1;
        font-size: .22rem;
        top: .1rem;
        left: .05rem;
        width: .4rem;
      }
    }
    .main {
      flex: 1;
      text-align: center;
      margin-right: -0.1rem;
    }

    .input {
      display: block;
      width: 100%;
      text-align: center;
      background: #e8e8e8;
      height: .29rem;
      font-size: .13rem;
      padding: 0 0.2rem;
      color: #333;
      /*margin: .08rem 0;*/
      border-radius: .1rem .1rem .1rem .1rem;
      margin: 0rem .1rem .05rem .1rem;
      line-height: .29rem;
    }


  }
  .top-nav {
    background-color: #fff;
    width: 100%;
    height: .45rem;
    display: flex;
    padding: 0 .1rem;
    justify-content: center;
    z-index: 2;
    li {
      display: block;
      color: #999999;
      font-size: .15rem;
      line-height: .45rem;
      flex: 1;
    }
    li.selected{
      border-bottom:1px solid #6C6C6C;
    }
  }
  .scroll-view {
    position: relative;
    top: 0;
    .scroll-view(100%);
    width: 100%;
    padding-bottom: .95rem;
    z-index: -1;
  }

  .banner {
    width: 100%;
    height: 2.25rem;
    img {
      width: 100%;
      height: 100%;
    }
  }

  .goods {
    text-align: left;
    padding: 0 .16rem;
    .goods__title {
      font-size: .17rem;
      color: #000000;
      text-align: center;
      width: 100%;
      height: .45rem;
      line-height: .45rem;
    }
    .goods__item {
      margin-bottom: .12rem;
      .goods__pic {
        width: 100%;
        height: 3.4rem;
        img {
          width: 100%;
          /*height: 3.4rem;*/
          height: 100%;
        }
      }

      h3 {
        font-size: .16rem;
        font-weight: 800;
        color: #333333;
        line-height: .24rem;
        margin-top: .13rem;
      }
      p {
        font-size: .14rem;
        .text-overflow(2);
        margin-bottom: .12rem;
      }
      .buy {
        display: flex;
        font-size: .14rem;
        justify-content: space-around;

        span {
          text-align: left;
          flex: 1;
          font-size: .12rem;
          color: #e75058;
        }
        div {
          text-align: right;
          border: 1px solid #666;

        }
      }
    }
  }
  .bottom-img {
    width: 100%;
    padding: 0 .16rem;
    height: auto;
  }
  .bottom-pic {
    width: 100%;
    height: 100%;
    position: relative;
    top: -0.6rem;
  }

</style>
