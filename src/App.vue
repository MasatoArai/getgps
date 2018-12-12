<template>
  <div id="app">
    <button type="button"  class="button is-medium is-dark" @click="open">現在地を取得しMapfunで見る</button>
      <b-input v-model="gatLongLat" placeholder="緯度経度の自動取得" disabled></b-input>
    <modal name="example">
      <div class="basic-modal">
         <b-notification type="is-danger" v-if="failGps">
            GPS情報の取得に失敗しました
        </b-notification>
        <section v-show="!failGps">
        <iframe id="mm" src=""></iframe>
        <div id="show_result">あなたの居る座標は{{showResult}}</div>
            <button type="button"  class="button is-primary" @click="googlemap">googleMAPで見る[別窓]</button>
        </section>
        <button class="button is-dark" type="button" @click="close">Close</button>
          
      </div>
        
    </modal>
         <b-notification type="is-danger" v-if="!canGeolocation">
            GPS情報の取得機能がありません。
        </b-notification>
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'

    
export default {
  name: 'app',
    data:function(){
        return{
            canGeolocation:true,
            long:0.0,
            lat:0.0,
            failGps:false,
            gatGps:false
        }
    },
    computed:{
        showResult:function(){
            return `緯度:${this.long},経度:${this.lat}`;
        },
        gatLongLat:function(){
           return this.gatGps?`${this.long},${this.lat}`:'';
        }
    },
  components: {
   // HelloWorld
  },
    methods: {
        googlemap () {
            let line = `https://www.google.com/maps/@${this.long},${this.lat}`;
            window.open(line);
        },
        open () {
            if(navigator.geolocation){
                this.canGeolocation=true;
            }else{
                this.canGeolocation = false;
                return false;
            }
            
          this.$modal.push('example');
            let self = this;
            navigator.geolocation.getCurrentPosition(successCallback, errorCallback);
            function successCallback(position) {
                self.lat = position.coords.latitude;
                self.long = position.coords.longitude;
                document.getElementById('mm').src="https://mapfan.com/map?c="+position.coords.latitude+","+position.coords.longitude+",15";
                    self.failGps = false;
                self.gatGps = true;
            }
                function errorCallback(e){
                    self.failGps = true;
                    console.log(e);
                }
        },
        close () {
          this.$modal.pop()
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
    }
    #mm{
        width: 98vw;
        height: 70vh;
        border: 1px solid black;
    }
    .modal-content {
    position: absolute;
    top: 0 !important;
    height: 100vh !important;
    left: 0 !important;
    right: 0 !important;
    opacity: 1;
    margin: 0 auto !important;
    padding: 1vw !important;
    width: 100vw !important;
    max-height: 100vh !important;
    background-color: #fff;
    -webkit-transform: translateY(0);
    transform: translateY(0);
    }
</style>
