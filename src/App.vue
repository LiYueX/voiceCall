<template>
  <div id="app" v-cloak>
    <el-card class="begin box-card">
        <h2>智能语音点名</h2>
        <p>当前人数 <strong>{{names.length}}</strong></p>
        <div>输入抽点人数：<input type="text" :placeholder='placeholder' class="namesNum" v-model='namesNum' @keyup='reg'></div>
        <el-button type="success" @click='beginCall' class='beginCall' :disabled="enabled1">开始点名</el-button>
        <el-button type="primary" @click='continueCall' class='beginCall' :disabled="enabled2">继续抽查</el-button>
        <el-button type="info" @click='rePlay' >重新播放</el-button>
        <el-button type="warning" @click='reCall' class='beginCall'>重新点名</el-button>
    </el-card>

    <el-card class="call box-card" v-show='vShow'>
        <el-popover
          ref="popover1"
          placement="top"
          title="点击姓名可重新播放"
          width="100"
          trigger="hover">
        </el-popover>
        <h1 @click='again'  v-popover:popover1 >{{name}}</h1>
        <div class="come" v-show='!over'>
          <el-button type="success" @click='coming' icon='circle-check' class='coming' :disabled="enabled">来了</el-button>
          <el-button type="danger" @click='noComing' icon='circle-close' class='noComing' :disabled="enabled">没来</el-button>
        </div>
        <el-button type="primary" v-show='over' icon='information' class='over'>点名结束</el-button>
        <audio :src="audioSrc" autoplay></audio>
    </el-card>

    <el-card class="call box-card" v-show='noComeName.length>0'>
      <h2>当然是选择原谅TA们喽~</h2>
      <ul>
        <li v-for='(item,index) in noComeName' :key='index' v-text='item'></li>
      </ul>
    </el-card>

  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      names:["石根厚","石建军","石正厚","刘占宽","刘耀清","贾六","石俊梅","刘飞","刘文明","高文利","高四孬","高文军","袁富存","尚文忠","贾乐","石栓柱","李龙","李海龙","王露","王桂香","王焕树","王二兵","韩凤英","王有罕","王山崎","杜茂仁","刘海欢","康桃花","杜元晓","张海宝","高建明","张飞雄","刘御","张润佘","张永强","郭三梅","杜云","魏红","刘继牛","郭武莱","高桂清","杜永强","杜文渊","高二媛","王二锁","王志宏","李金云","李欣媛","戴双宝","王焕锁","孙瑞军","石飞鹏","石胜厚","康振清","王禅","高志新","刘飞云","刘二云","刘三正","刘东","刘双正","namesNum","vShow","noComeName"],
      namesNum:'',
      name:'点击姓名可以重播哦',
      vShow:true,
      noComeName:[],
      callNames:[],
      srcs:[],
      audioSrc:'',
      activeIndex:0,
      over:false,
      enabled:true,
      enabled1:true,
      enabled2:true,
      placeholder:''
    }
  },
  mounted(){
    this.placeholder = '请输入数字1~'+this.names.length
  },
  methods:{
    beginCall(){//开始点名
      this.over=false
      this.enabled=false
      this.enabled1=true
      this.enabled2=true
      this.rand();
    },
    continueCall(){//继续抽查
      this.over=false
      this.callNames=[]
      this.srcs=[]
      this.activeIndex=0
      this.enabled=false
      this.enabled1=true
      this.enabled2=true
      this.rand();
    },
    rePlay(){
      this.activeIndex=0
      this.audioSrc=this.srcs[0]
      this.name=this.callNames[0]
      this.over=false
      this.noComeName=[];
      this.again();
    },
    rand(){//随机抽取人名
      var len = this.names.length;
      if(len<Number(this.namesNum)){
        this.namesNum=len;
      }
      for(let i=0;i<Number(this.namesNum);i++){
        let temp = Math.floor(Math.random() * this.names.length) | 0;
        let value = this.names.splice(temp,1)[0];
        let src='';
        if(value.match(/[a-zA-Z]+$/)){
          src = "http://fanyi.baidu.com/gettts?lan=en&text="+value+"&spd=3&source=web";
        }else{
          src = "http://fanyi.baidu.com/gettts?lan=zh&text="+encodeURI(value)+"&spd=5&source=web";
        }
        this.callNames.push(value)
        this.srcs.push(src)
      }
      this.audioSrc=this.srcs[0]
      this.name=this.callNames[0]
    },
    reg(){
      if(!(/^[0-9]*$/g.test(this.namesNum)) || this.namesNum>this.names.length){
        this.namesNum=this.names.length
      }else if(Number(this.namesNum)<=0){
        this.namesNum=this.names.length
      }
      console.log(this.callNames.length)
      if(this.callNames.length==0){
        this.enabled1=false
      }else{
        this.enabled1=true
      }
      this.enabled2=false
    },
    reCall(){//重新点名
      window.location.reload()
    },
    coming(){//来了
      this.nextStep()
    },
    noComing(){//没来
      this.noComeName.push(this.name);
      this.nextStep()
    },
    again(){//语音重放
      var audio = document.getElementsByTagName('audio')[0];
      audio.load();
    },
    nextStep(){//下一个名字
      this.activeIndex++
      this.audioSrc=this.srcs[this.activeIndex]
      this.name=this.callNames[this.activeIndex]
      if(this.activeIndex == this.namesNum && this.noComeName.length==0){
        this.name='じò ぴé 今天人员全部到齐，漂亮！'
        this.over=true;
        this.enabled1=true;
        this.enabled2=false;
        this.audioSrc = "http://fanyi.baidu.com/gettts?lan=zh&text="+encodeURI('今天人员全部到齐，漂亮！')+"&spd=5&source=web"
      }
      if(this.activeIndex == this.namesNum && this.noComeName.length!=0){
        this.name='点名结束，让我逮到了吧└(^o^)┘'
        this.over=true;
        this.enabled1=true;
        this.enabled2=false;
        this.audioSrc = "http://fanyi.baidu.com/gettts?lan=zh&text="+encodeURI('点名结束,今天未到的人有：'+this.noComeName)+"&spd=5&source=web"
      }
      this.again();
    }
  }
}
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    font-size: 18px;
  }

  .begin,.call{
    width: 640px;
    height: 280px;
    margin: 0 auto;
  }
  .beginCall{
    margin-top: 30px;
  }
  .namesNum{
    width: 150px;
    font-size:18px;
    line-height: 1.5;
    padding-left: 10px;
    border-radius: 4px;
    outline: none;
    border: none;
    border: 1px solid #222;
  }
  .come,.over{
    margin-top: 80px;
  }
  .call h1{
    cursor:pointer;
  }
  h2{
    margin: 0;
    padding: 0;
    margin-bottom: 20px;
  }
  ul{
    padding: 0;
    margin: 0;
    overflow: hidden;
  }
  .call li{
    float: left;
    list-style: none;
    padding: 0;
    margin-right: 10px;
    line-height: 40px;
  }
</style>
