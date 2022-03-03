<template>
  <div id="app" class="container">
    <h1>業務委託者検索</h1>
    <div v-if="multi_flg">
      <form @submit.prevent="search_multi">
        <div class="fullname">
          検索する氏名（姓名）<br/>
          <textarea v-model="fullname" placeholder="例）大分 太郎" cols="20" rows="10"></textarea><br/>
          <input type="submit" value="search" class="btn">
          <p class="s-mess">姓と名の間に全角スペースを入れてください。完全一致で検索します。</p>
          <p class="s-mess">複数入力する場合は1行に1名を入力してください。</p>
          <p class="s-mess">漢字の違いや外国名の場合に検索されない場合があります。</p>
        </div>
      </form>
    </div>
    <div v-else>
      <form @submit.prevent="search">
        <div class="fullname">
          検索する氏名（姓名） : <input type="text" v-model="fullname" placeholder="例）大分 太郎" required>
          <input type="submit" value="search" class="btn">
          <p class="s-mess">姓と名の間に全角スペースを入れてください。完全一致で検索します。</p>
        </div>
      </form>
    </div>
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
      item_id_firstname: '',
      multi_flg: true
    }
  },
  watch:{
    token: {
      handler: function(){
        sessionStorage.setItem('formifytoken', this.token);
      }
    },
    fullname: function(newValue){
      if ( newValue.length > 1024 ){
         this.fullname = newValue.slice(0,1024);
         alert('検索する人数を減らしてください（最大100名程度）');
      }else{
         this.fullname = newValue;
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
    split_name: function(fullname_){
      var _name = fullname_.split('　');
      if(_name.length < 2) {
         _name = fullname_.split(' ');
      }
      this.last_name = _name[0];
      _name.shift();
      this.first_name = _name.join('　');
      if(!this.last_name){
         this.message = '姓を入力してください(' + fullname_ + ')';
         this.first_name = '';
         return false;
      }
      if(!this.first_name){
         this.message = '空白で区切って名を入力してください(' + fullname_ + ')';
         this.last_name = '';
         return false;
      }
      return true;
    },
    search_multi: function(){
      this.checkToken(true);
      var fullnames = this.fullname.split('\n');
      var firstnames = new Array();
      var lastnames = new Array();
      for (let i=0; i < fullnames.length; i++){
        var fullname_ = fullnames[i].trim();
        if (fullname_.length < 1){
          continue;
        }
        if(!this.split_name(fullnames[i])){
          return;
        }
        firstnames.push(this.first_name);
        lastnames.push(this.last_name);
      }
      var params = {status: 'approved', category_id: this.category_id, [this.item_id_firstname] : firstnames, [this.item_id_lastname] : lastnames};
      var qparam = new URLSearchParams(params);
      var requestOptions = {
         method: "GET",
         headers: { "Authorization": "Bearer " + this.token,
             "Content-Type" : "application/html" }
      };
      this.loading=true;
      this.initValue();
      fetch(this.base_url + '/full_list/?' + qparam, requestOptions)
          .then(response => {
            if (response.status == '401'){
              this.messages = 'Timeout. Please research.';
              this.token = '';
              this.getToken();
              alert("Timeout. Please push search again.");
              return;
            }
            return response.text();
          })
          .then(data => this.html = data)
          .catch(error => {
            console.log(error);
            this.message = 'エラーが発生しました。';
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
