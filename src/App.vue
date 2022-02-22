<template>
  <div id="app" class="container">
    <h1>業務委託者検索</h1>
    <form @submit.prevent="search">
      <div class="fullname">
        検索する氏名（姓名） : <input type="text" v-model="fullname" placeholder="例）大分 太郎" required>
        <input type="submit" value="search" class="btn">
        <p class="s-mess">姓と名の間に全角スペースを入れてください。完全一致で検索します。</p>
      </div>
    </form>
    <div v-if="message" class="message">{{ message }}</div>
    <div v-if="loading" class="container" id="loading">
      <img src="./assets/loading-19.gif" alt="loading">
      しばらくお待ちください
    </div>
    <div class="container" v-html="html"></div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data(){
    return {
      first_name : '',
      last_name : '',
      fullname : '',
      token: '',
      html: '',
      loading: false,
      message: '',
      base_url: '',
      category_id: '',
      item_id_lastname: '',
      item_id_firstname: ''
    }
  },
  watch:{
    token: {
      handler: function(){
        sessionStorage.setItem('formifytoken', this.token);
      }
    }
  },
  methods:{
    initValue: function(){
      this.first_name = '';
      this.last_name = '';
      this.html = '';
      this.message = '';
    },
    checkToken: function(mess){
      this.token = sessionStorage.getItem('formifytoken');
      if(this.token){
        return;
      }
      if(mess){
        alert("セッションが終了しました");
      }
      this.getToken();
    },
    getToken: function(){
      var username = process.env.VUE_APP_TOKEN_ID;
      var password = process.env.VUE_APP_TOKEN_PASSWD;
      var requestOptions = {
        method: "POST",
        headers: { "Content-Type":"application/json"},
        body: JSON.stringify({auth: {signin_name: username, password: password}})
      };
      fetch(this.base_url + '/user_token', requestOptions)
        .then(response => response.json())
        .then(data => {this.token = data.jwt;})
    },
    search: function(){
      this.checkToken(true);
      this.fullname.trim();
      var _name = this.fullname.split('　');
      if(_name.length < 2) {
         _name = this.fullname.split(' ');
      }
      this.last_name = _name[0];
      _name.shift();
      this.first_name = _name.join('　');
      if(!this.last_name){
         this.message = '姓を入力してください';
         this.first_name = '';
         return;
      }
      if(!this.first_name){
         this.message = '名を入力してください';
         this.last_name = '';
         return;
      }
      var params = {status: 'approved', category_id: this.category_id, [this.item_id_firstname] : this.first_name, [this.item_id_lastname] : this.last_name};
      var qparam = new URLSearchParams(params);
      var requestOptions = {
         method: "GET",
         headers: { "Authorization": "Bearer " + this.token,
             "Content-Type" : "application/html" }
      };
      this.loading=true;
      this.initValue();
      fetch(this.base_url + '/full_list/?' + qparam, requestOptions)
          .then(response => response.text())
          .then(data => this.html = data)
          .then(console.log(this.html))
          .catch(error => {
              console.log(error);
              this.message = '検索に失敗しました';
          })
          .finally(() => this.loading = false)
    }
  },
  created(){
    this.base_url = process.env.VUE_APP_BASE_URL;
    this.category_id = process.env.VUE_APP_CATEGORY_ID;
    this.item_id_lastname = 'item_id_' + process.env.VUE_APP_ITEM_ID_LASTNAME;
    this.item_id_firstname = 'item_id_' + process.env.VUE_APP_ITEM_ID_FIRSTNAME;
    this.checkToken(false);
    this.initValue();
    this.token = sessionStorage.getItem('formifytoken');
  }
}
</script>

<style>
@import "./assets/styles.css";
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
