<template>
  <div class="container" v-if="status">
    <div class="columns is-flex-wrap-wrap">
      <div class="column is-6" v-for="item in list" :key="item.ApiKey">
        <div class="box">
          <div class="content">
            <div class="level">
              <h5 class="title level-left mb-0">{{item.PcName}}</h5>
              <span class="level-right icon-text has-text-grey-light">
                <span class="icon">
                  <font-awesome-icon :icon="['fab', item.OS]" fixed-width />
                </span>
                <span class="has-text-grey-light">{{item.OS === "windows"? "Windows":"Linux"}}</span>
              </span>
            </div>
            <div class="columns is-size-6 is-mobile">
              <div class="column is-6">
                <span class="icon va mr-1">
                  <font-awesome-layers class="fa-lg">
                    <font-awesome-icon icon="circle" :style="{color:(item.Online?'#23d160':'#7a7a7a')}" />
                    <font-awesome-icon :icon="item.Online ? 'check' : 'times'" transform="shrink-6" style="color: white" />
                  </font-awesome-layers>
                </span>
                <span :class="item.Online ? 'is-size-6 has-text-primary' : 'is-size-6 has-text-grey'">{{item.Online ? "矿机在线" : "矿机离线"}}</span>
              </div>
              <div class="column is-6">
                <span class="icon va mr-1">
                  <font-awesome-layers class="fa-lg">
                    <font-awesome-icon icon="circle" :style="{color:(item.Online?'#23d160':'#f14668')}" />
                    <font-awesome-icon :icon="item.Online ? 'check' : 'times'" transform="shrink-6" style="color: white" />
                  </font-awesome-layers>
                </span>
                <span :class="item.Online ? 'is-size-6 has-text-primary' : 'is-size-6 has-text-danger'">{{item.Online ? "正在耕种" : "停止耕种"}}</span>
              </div>
            </div>
            <div class="columns is-multiline is-mobile">
              <div class="column is-12">
                <span class="icon va mr-1">
                  <font-awesome-layers class="fa-lg">
                    <font-awesome-icon icon="circle" :style="{color:(item.Online?'#23d160':'#febb00')}" />
                    <font-awesome-icon :icon="item.Online ? 'check' : 'times'" transform="shrink-6" style="color: white" />
                  </font-awesome-layers>
                </span>
                <span :class="item.Online ? 'color is-size-6 has-text-primary' : 'color is-size-6 has-text-warning'">{{item.Online ? "检图正常" : "检图异常"}}</span>
              </div>
              <div class="column is-6 has-text-grey-dark">
                实际图数：{{item.PlotNumber}}
              </div>
              <div class="column is-6 has-text-grey-dark">
                检图总数：{{item.PlotInfo.PlotNumber}}
              </div>
            </div>
            <hr class="mt-0 mb-3" />
            <div class="columns is-mobile">
              <div class="column is-3 pt-5">CPU使用率</div>
              <div class="column is-9 has-text-centered">
                <span>{{item.CPU.LoadPerc}}%</span>
                <progress class="progress is-primary is-small" :value="item.CPU.LoadPerc" max="100">{{item.CPU.LoadPerc}}</progress>
              </div>
            </div>
            <div class="columns is-mobile">
              <div class="column is-3 pt-5">内存使用率</div>
              <div class="column is-9 has-text-centered">
                <div class="level mb-0">
                  <span class="level-left mb-0">已用：{{item.Memory.Used}}</span>
                  <span class="level-right mb-0">总数：{{item.Memory.TotalMemory}}</span>
                </div>
                <progress class="progress is-warning is-small" :value="item.Memory.MemPerc" max="100">{{item.Memory.MemPerc}}</progress>
              </div>
            </div>
            <div v-if="item.Hidden">
              <hr />
              <div class="columns mb-0 is-size-7 is-mobile">
                <div class="column is-3 pt-3">最后活跃时间：</div>
                <div class="column is-9">{{item.time}}</div>
              </div>
              <div class="columns mb-0 is-size-7 is-mobile">
                <div class="column is-3 pt-3">CPU：</div>
                <div class="column is-9">{{item.CPU.CpuName}}</div>
              </div>
              <div class="columns mb-0 is-size-7 is-mobile">
                <div class="column is-3 pt-3">IP：</div>
                <div class="column is-9">{{item.NetWorks.IPAddress}}</div>
              </div>
              <div class="columns is-size-7 is-mobile">
                <div class="column is-3 pt-3">MAC：</div>
                <div class="column is-9">{{item.NetWorks.MACAddress}}</div>
              </div>
              <div class="columns is-flex-wrap-wrap is-mobile" v-if="item.Drive.length > 0">
                <div class="column is-3 is-size-7" v-for="(d,index) in item.Drive" :key="index">
                  <div class="is-size-6">{{d.DeviceID}}</div>
                  <div class="is-flex is-flex-wrap-wrap is-justify-content-space-between	">
                    <div class="is-flex">已用:{{d.Used}}</div>
                    <div class="is-flex">容量:{{d.Size}}</div>
                  </div>
                  <div>
                    <span>图数:{{d.PlotNumber}}</span>
                  </div>
                  <div>
                    <span>扫描用时:{{d.ScanTime}}</span>
                  </div>
                  <progress class="progress is-danger is-small" :value="d.Percent" max="100">{{d.Percent}}</progress>
                </div>
              </div>
            </div>
            <hr />
            <button class="button is-small is-link is-fullwidth" @click="HiddenShow(item.ApiKey)">
              <span>{{item.Hidden?"点击收起详细":"点击展开详细"}}</span>
              <span class="icon">
                <font-awesome-icon :icon="item.Hidden ? 'angle-double-up' : 'angle-double-down'" fixed-width />
              </span>
            </button>
          </div>
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
    const ip = "a.chiafm.xyz:13002"
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
        if (this.list.length === 0) {
          for (let i = 0; i < content.length; i++) {
            t.list = [...t.list, JSON.parse(content[i])];
            if (i === content.length -1) {
              t.status = true
            }
          }
        }
      }else {
        if (content === "offline") {
          const sender = received_msg.sender.split("_")[1]
          for (let i = 0; i < t.list.length; i++) {
            if (t.list[i].ApiKey === sender) {
              t.list[i].Online = false
            }
          }
        }else{
          if (t.list.length <= 0) {
            t.list = [...t.list,content]
            t.status = true
          }
          for (let i = 0; i < t.list.length; i++) {
            if (t.list[i].ApiKey === content.ApiKey) {
              t.list[i] = content
            }
          }
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
      this.status = false
      console.log('断开连接',e);
    },
    checkHas(arr,ApiKey){
      arr = arr.filter((x)=>{
        return x.ApiKey === ApiKey
      })
      // console.log(arr)
      return arr.length
    },
    HiddenShow(v){
      for (let i = 0; i < this.list.length; i++) {
        if (this.list[i].ApiKey === v) {
          this.list[i].Hidden = !this.list[i].Hidden
        }
      }
    }
  },
  unmounted() {
    this.websock.close()
    console.log('连接已关闭...')
  }
}
</script>

<style>
#app {
  padding-top: 1rem
}
.va {
  vertical-align: bottom
}
.color.has-text-warning {
  color: #d79e00 !important;
}
</style>
