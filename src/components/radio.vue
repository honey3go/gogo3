<template>
  <div class="playlist main">
    <div  v-show="hasResult">
      <div class="playlist-left">
        <div class="playlist-head">
          <div class="playlist-cover radio-cover">
            <img src="http://p1.music.126.net/F-j4xp43rDIwGTs5mFK9qw==/18678503533108448.jpg?param=140y140">
            <span class="radio-wrap"></span>
          </div>
          <div class="playlist-content radio-content">
            <div class="content-title" id="radiotitle">
              <i id="radiotag"></i>
              <h2>主播子林：友谊如何才能保持长久？ - 梁实秋</h2>
            </div>
            <div class="radio">
              <i class="radio-pic"></i>
              <a href="" class="radio-link">十点读书</a>
              <a href="javascript:;" class="radio-pre">
                <i>
                  <em></em>
                  订阅
                </i>
              </a>
            </div>
          </div>
          <div class="content-opreation">
              <a href="#" class="btn-play radio-play">
                <i><em class="ply"></em>播放</i>
              </a>
              <a href="#" class="btn-fav">
                <i>订阅</i>
              </a>
              <a href="#" class="btn-share">
                <i>分享</i>
              </a>
              <a href="#" class="btn-dl">
                <i>下载</i>
              </a>
              <a href="#" class="btn-cm">
                <i>评论</i>
              </a>
              <div class="clear"></div>
          </div>
          <div class="radio-info">
            <strong>十点读书 第712期</strong>
            <i>2017-04-12创建</i>
            <i>播放：<em>122223</em>次</i>
          </div>
          <pre v-show="!songs.isShowMore"><b class="u-desc">介绍：</b>hhhhh</pre>
        </div>
        <div class="radio-song">
          <div class="radio-songhead">
            <strong>节目包含歌曲列表</strong>
            (2首歌)
          </div>
        </div>
        <comment :url="cmtUrl"></comment>
      </div>
      <playlistRight></playlistRight>
    </div>
    <div class="loading" v-show="!hasResult">
      <i></i>
      加载中...
    </div> 
  </div>
</template>

<script>
import playlistRight from './generalRight'
import comment from './generalComment'
import { mouseBtnEv } from '../js/generalChangeVal.js'

export default {
  name: 'playlist',
  components:{
    playlistRight,comment
  },
  data () {
    return {
      hasResult:false,//是否返回歌单数据
      songs:{//歌单
        coverImgUrl:null,
        name:null,
        subscribedCount:null,
        shareCount:null,
        commentCount:null,
        tags:null,
        trackCount:null,
        playCount:null,
        tracks:null,     //歌曲
        creator:{},    //歌单创建者
        createtime:null, //歌单创建时间
        descDot:null,    //歌单介绍part1
        descMore:null,   //歌单介绍part2
        isShowMore:false,//歌单介绍是否展开
      },
      listUrl:`http://123.206.211.77:33333/api/v1/playlist/detail/866169651`,
      cmtUrl:`http://123.206.211.77:33333/api/v1/playlist/comments/866169651/page/`,
    }
  },
  methods:{
    //歌单介绍-展开/收起按钮点击事件
    tabShowMore:function(){
      this.songs.isShowMore = !this.songs.isShowMore;
    },
    //歌单播放按钮点击事件
    plyClick:function(index){
      var clickList = this.songs.tracks.map(function(item){
        return item.click;
      });

      var current = clickList.indexOf(true);
      if (current>-1){
        mouseBtnEv.setNewVal(this.songs.tracks[current], 'click', false);
      }
      mouseBtnEv.setNewVal(this.songs.tracks[index], 'click', true);
    },
    //歌单播放按钮点击事件代理
    plySong:function(ev){
      var ev = ev||window.event;
      var target = ev.target||ev.srcElement;

      if (target.nodeName.toLowerCase() == "span"){
        var index = parseInt(target.dataset.tag);
        this.plyClick(index);             
      }
    },
  },
  created:function(){
    //请求歌单数据
    this.$http.get(this.listUrl)
      .then(response => {
        this.hasResult = response.data.result;//初始化全部歌单数据
      })
      .catch(response => {
        console.log(response)
    });
  },
  watch:{
    //歌单数据返回后，提取、格式化需要的数据
    hasResult:function(result){
      //解构result.tracks
      var originTracks = result.tracks,
          list = new Array();
      for ( let item of originTracks ){ 
        let { id,duration, name:songName, album:{name:albName}, album:{id:albId}, album:{artists:[{name:artName}]}, album:{artists:[{id:artId}]}} = item;
        duration = mouseBtnEv.changeTime(duration);
        list.push({ id, duration, songName, albName, albId, artName, artId, click:false});
      }
      //更改页面title
      document.title = `${result.name} - 网易云音乐`;
      //初始化songs
      this.songs = {
        coverImgUrl:result.coverImgUrl,
        name:result.name,
        subscribedCount:result.subscribedCount,
        shareCount:result.shareCount,
        commentCount:result.commentCount,
        tags:result.tags,
        trackCount:result.trackCount,
        playCount:result.playCount,
        tracks:list,
        creator:result.creator,
        createtime:new Date(result.createTime).toLocaleDateString().replace(/\//g,"-"),
        descDot:result.description.substr(0,99),
        descMore: result.description.length>99?result.description:null,
        isShowMore:false,   
      };
    },
  }
}
</script>

<style>
.radio-songhead{
  height: 32px;
  padding:0 10px; 
  line-height: 33px;
  background-color: rgb(247,247,247);
}
.radio-song{
  margin-top: 27px;
  border: rgb(217,217,217) solid 1px;
  font-size: 12px;
}
.radio-info strong{
  float: left;
  font-size: 14px;
  color: black;
}
.radio-info i{
  float: left;
  margin-left: 20px;
}
.radio-info em{
  font-weight: bold;
  font-style: normal;
  color: #c20c0c;
}
.radio-info{
  margin: 20px 0;
  font-size: 12px;
  line-height: 20px;
  color: rgb(153,153,153);
  overflow: hidden;
}
.radio-play{
  margin-right: 6px;
}
.radio-pre{
  float: left;
  height: 28px;
  margin-left: 17px;
  padding-right: 5px;
  background: url(../assets/button2.png) no-repeat scroll 100% -2400px;
}
.radio-pic{
  margin: 5px 7px 0 0;
}
.radio-pre i{
  line-height: 29px;
  padding: 0 10px;
  background: url(../assets/button2.png) no-repeat scroll 0 -2370px;
}
.radio-pre em{
  float: left;
  width: 14px;
  height: 14px;
  margin: 7px 4px 0 0;
  background: url(../assets/icon2.png) no-repeat scroll -50px 0;
}
.radio-link{
  float: left;
  margin-left: 10px;
  font-size: 16px;
}
.radio-link:hover{
  text-decoration: underline;
}
.radio-pic{
  float: left;
  width: 16px;
  height: 17px;
  background: url(../assets/icon2.png) no-repeat scroll -50px -20px;
}
.radio{
  margin: 40px 0;
  overflow: hidden;
  line-height: 29px;
}
#radiotag{
  width: 73px;
  margin-left: -83px;
  background: url(../assets/icon2.png) no-repeat scroll 0 -75px;
}
#radiotitle{
  position: relative;
  margin-left: 83px;
}
.radio-cover{
  width: 140px;
  height:140px;
}
span.radio-wrap{
  width: 148px;
  height:148px;
}
.radio-content{
  width: 470px;
}
</style>
