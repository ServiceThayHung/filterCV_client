<template>
  <div id="app">
    <div class="container-item">
      <h1>Chọn tham số:</h1>
      <span>Chọn file: </span><input type="file" multiple="multiple"><br>
      <span>Nhập các tiêu chí cách nhau bởi dấu phẩy: </span>
      <br>
      <textarea v-model="criteria" style="width: 80%"></textarea>
      <br>
      <span>Gửi email cho các ứng viên: </span><input type="checkbox" v-model="isSendMail">
      <br>
      <button v-on:click="execute" >Thực hiện</button>
    </div>
    <div class="container-item" style="flex: 2"></div>
    <div class="container-item">
      <h1>Danh dách các tiền trình xử lý: </h1>
      <div v-for="(item, index) in listProcess" :key="index" class="proccess">
        <div v-if="item.step != 5" class="spinner-border text-info process-running-custom" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
        <i class="bi bi-check"></i>
        <div v-if="item.step == 5">
          <svg xmlns="http://www.w3.org/2000/svg" width="50" height="50" fill="#14cd2a" class="bi bi-check complete-custom" viewBox="0 0 16 16">
            <path d="M10.97 4.97a.75.75 0 0 1 1.07 1.05l-3.99 4.99a.75.75 0 0 1-1.08.02L4.324 8.384a.75.75 0 1 1 1.06-1.06l2.094 2.093 3.473-4.425a.267.267 0 0 1 .02-.022z"/>
          </svg>
</div>
        <div class="info-process">
          <div>{{item.step}}/5</div>
          <div>{{item.name}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  },
  created()
  {
    let me = this;
    console.log(me);

  },
  mounted() {
    let me = this;
    console.log(me);

    me.webSocket = new WebSocket("wss://localhost:44373/");

    me.webSocket.onopen = function () {
            
    };
    me.webSocket.onclose = function () {
        
    };
    me.webSocket.onmessage = function (event) {
      me.handleMessage(event);
    };

  },
  methods: {
    execute()
    {

    },
    handleMessage(event)
    {
      let info = event.data.split('|');
      console.log(info);
      let me = this;
      let isExistProccess = false;
      me.listProcess.forEach(proccess => {
        if(proccess.id == info[0])
        {
          proccess.step = info[1];
          proccess.name = info[2];
          isExistProccess = true;
        }
      });
      if(!isExistProccess)
      {
        me.listProcess.push(
          {
            id: info[0],
            step: info[1],
            name: info[2],
          }
        );
      }
    }
  },
  data(){
    return {
      criteria: '',
      isSendMail: false,
      listProcess: [
        {
          id: '6e49785a-f55f-11ec-b939-0242ac120002',
          step: 2,
          name: 'Upload file server',
        },
        {
          id: '6e49785a-f55f-11ec-b939-0242ac120002',
          step: 5,
          name: 'Upload file server',
        },
      ]
    }
  }
}
</script>

<style>

@import url('./assets/css/app.css');

#app {
  display: flex;
  width: 100vw;
  height: 100vh;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
