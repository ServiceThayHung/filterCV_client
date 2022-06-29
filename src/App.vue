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
      <button @click="execute">Thực hiện</button>
    </div>
    <div class="container-item" style="flex: 2">
      <div class="data-show" v-if="displayProcess.data != null">
        <h1>Danh sách CV có từ khóa</h1>
        <table style="width: 100%">
          <thead>
            <tr>
              <th>STT</th>
              <th>Tên CV</th>
              <th>Số từ khóa khớp</th>
              <th>%khớp</th>
              <th>Các tiêu chí khớp</th>
            </tr>
          </thead>
          <tbody>
             <tr v-for="(item, index) in displayProcess.data.suitableList" :key="index">
              <td>{{index + 1}}</td>
              <td>{{item.name}}</td>
              <td>{{item.numberOfMatched}}</td>
              <td>{{item.matchPercent}}</td>
              <td>{{item.listMatched.join(',')}}</td>
            </tr>
          </tbody>
        </table>

        <h1 style="padding-top: 100px">Danh sách CV không chứa từ khóa</h1>
        <table style="width: 100%;">
          <thead>
            <tr>
              <th>STT</th>
              <th>Tên CV</th>
              <th>Số từ khóa khớp</th>
              <th>%khớp</th>
              <th>Các tiêu chí khớp</th>
            </tr>
          </thead>
          <tbody>
             <tr v-for="(item, index) in displayProcess.data.unsuitableList" :key="index">
              <td>{{index + 1}}</td>
              <td>{{item.name}}</td>
              <td>{{item.numberOfMatched}}</td>
              <td>{{item.matchPercent}}</td>
              <td>{{item.listMatched.join(',')}}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div class="container-item">
      <h1>Danh dách các tiền trình xử lý: </h1>
      <div v-for="(item, index) in listProcess" :key="index" class="proccess" @click="viewResult(item)">
        <div v-if="item.step != 'final'" class="spinner-border text-info process-running-custom" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
        <i class="bi bi-check"></i>
        <div v-if="item.step == 'final'">
          <svg xmlns="http://www.w3.org/2000/svg" width="50" height="50" fill="#14cd2a" class="bi bi-check complete-custom" viewBox="0 0 16 16">
            <path d="M10.97 4.97a.75.75 0 0 1 1.07 1.05l-3.99 4.99a.75.75 0 0 1-1.08.02L4.324 8.384a.75.75 0 1 1 1.06-1.06l2.094 2.093 3.473-4.425a.267.267 0 0 1 .02-.022z"/>
          </svg>
        </div>
        <div class="info-process">
          <div v-if="item.step != 'final'">{{item.step}}/7</div>
          <div v-if="item.step == 'final'">Complete</div>
          <div>{{item.name}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  components: {
  },
  created()
  {
    let me = this;
    me.baseUrl = 'https://filtercv-taskservice.herokuapp.com/api/FilterCV';
    
  },
  mounted() {
    let me = this;

    me.webSocket = new WebSocket("wss://filtercv-server.herokuapp.com/");

    me.webSocket.onopen = function () {
            
    };
    me.webSocket.onclose = function () {
        
    };
    me.webSocket.onmessage = function (event) {
      me.handleMessage(event);
    };
  },
  methods: {
    viewResult(record){
      let me = this;
      me.displayProcess = record;
      console.log(me.displayProcess);
    },
    execute()
    {
      let me = this;
      me.callApiFilterCV();
    },
    handleMessage(event)
    {
      let info = event.data.split('|');
      let me = this;
      let isExistProccess = false;

      if(info[1] == "final")
      {
        me.listProcess.forEach(proccess => {
          if(proccess.id == info[0])
          {
            proccess.step = info[1];
            proccess.name = info[2];
            proccess.data = JSON.parse(info[3]);
            isExistProccess = true;
          }

          if(me.displayProcess.id == proccess.id)
            me.displayProcess = proccess;
        });

        if(!isExistProccess)
        {
          me.listProcess.push(
            {
              id: info[0],
              step: info[1],
              name: info[2],
              data: JSON.parse(info[3])
            }
          );
        }
      }
      else
      {
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
              data: null
            }
          );
        }
      }

    },
    callApiFilterCV()
    {
      let me = this;
      var formData = new FormData();
      var eleInput = document.querySelector('input[type=file]');
      if(eleInput.files && eleInput.files.length > 0)
      {
        for(let i = 0; i< eleInput.files.length;i++)
        {
          formData.append("files", eleInput.files[i]);
        }
        formData.append("file_count", eleInput.files.length);
        formData.append("keyWord", me.criteria);
        formData.append("enable_notification", me.isSendMail);
        formData.append("urlUpdateProcess", "https://filtercv-server.herokuapp.com/api/proccess/update");

        axios.post(me.baseUrl, formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }).then(res => { console.log(res) })
      }
    }
  },
  data(){
    return {
      criteria: '',
      isSendMail: false,
      displayProcess: {
        id: '6e49785a-f55f-11ec-b939-0242ac120002',
        step: 2,
        name: 'Upload file server',
        data: null
      },
      listProcess: []
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
