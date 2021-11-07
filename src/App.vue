<template>
  <div class="container" v-if="status">
    <div class="columns is-multiline">
      <div class="column is-5" v-for="(item,index) in list" :key="item.ApiKey">
        <div class="box">
          <h3>{{item.PcName}}</h3>
          <div v-if="item.Online">{{index}}</div>
          <div>{{item.CPU.LoadPerc}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  // components: {
  //   ComputerList
  // },
  data(){
    return{
      websock: null,
      status: false,
      list: [],
    }
  },
  created(){
    console.log("start")
    const ip = "127.0.0.1:13002"
    const path = "server"
    const token = "manage_sdkjfskldjflksdjf"
    this.initWebSocket(ip,path,token)
  },
  methods: {
    initWebSocket(ip,path,token){ //初始化weosocket
      const wsuri = `ws://${ip}/api/${path}?token=${token}`;
      this.websock = new WebSocket(wsuri);
      this.websock.onmessage = this.websocketonmessage;
      this.websock.onopen = this.websocketonopen;
      this.websock.onerror = this.websocketonerror;
      this.websock.onclose = this.websocketclose;
    },
    websocketonerror(){//连接建立失败重连
      this.initWebSocket();
    },
    websocketonmessage(e){ //数据接收
      let t = this
      let content = "offline"
      let received_msg = JSON.parse(e.data);
      if (received_msg.content !== "offline") {
        content = JSON.parse(received_msg.content)
      }
      if (Array.isArray(content)) {
        console.log("array")
        for (let i = 0; i < content.length; i++) {
          const element = content[i];
          t.list = [...t.list, JSON.parse(element)];
          if (i === content.length -1) {
            t.status = true
          }
        }
      }else {
        if (content === "offline") {
          const sender = received_msg.sender.split("_")[1]
          t.list = t.list.filter((x) => {
            if (x.ApiKey == sender) {
              x.Online = false
            }
            return x
          })
        }else{
          if (t.list.length <= 0) {
            t.list = [...t.list,content]
            t.status = true
          }
          t.list = t.list.map((x) => {
            if (x.ApiKey === content.ApiKey) {
              x = content
            }
            return x
          })
          if (this.checkHas(t.list,content.ApiKey) === 0) {
            t.list.push(content)
          }
        }
      }
    },
    websocketsend(Data){//数据发送
      this.websock.send(Data);
    },
    websocketclose(e){  //关闭
      console.log('断开连接',e);
    },
    checkHas(arr,ApiKey){
      arr = arr.filter((x)=>{
        return x.ApiKey === ApiKey
      })
      console.log(arr)
      return arr.length
    }
  },
  unmounted() {
    this.websock.close()
    console.log('连接已关闭...')
  },
  watch:{
    list(val) {
      console.log(val)
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
 
.columns {
   flex-wrap: wrap; 
   justify-content: space-between
}
.column.is-5 {
height: 100%;
}
</style>
