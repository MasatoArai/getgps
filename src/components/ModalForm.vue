<template>
                <div class="modal-card">
                    <header class="modal-card-head">
                        <p class="modal-card-title" style="text-align:center">位置情報設定</p>
                    </header>
                    <section class="modal-card-body">
                        <input class="test" style="width:80%"></input>
                        <b-field label="住所検索" :message="addressBox.inFocus?'地域名をキーワードに住所を取得できます':''">
                            <div>
                                
                    <div class="control addresses has-icons-left is-clearfix"><input type="text" autocomplete="off" placeholder="住所入力" class="input test" v-model="addressBox.value" @focus="doForcus(addressBox,true)"  > <span class="icon is-left"><i class="mdi mdi-magnify mdi-24px"></i></span> </div>
                                <!--
                            <b-input
                                class="addresses" placeholder="住所入力"
                                     icon="magnify" v-model="addressBox.value" @focus="doForcus(addressBox,true)" @blur="doForcus(addressBox,false)" >
                            </b-input>-->
                            <button class="button is-primary is-small" @click="getGeocodeFromAddress(addressBox.value)">検索</button>
                            </div>
                        </b-field>

                        <b-field label="緯度経度情報"
                                 :type="latlongBox.correct?'':'is-danger'"
                                 :message="latLongBoxStatuses.labelMessage"
                                 >
                            <div>
                            <b-input
                                placeholder="緯度経度" v-model="latlongBox.value" icon="map-marker-radius" disabled class="latlng">
                            </b-input>
                            </div>
                        </b-field>
                        
                          <button class="button" :disabled="!hasGeolocation">
                            <b-icon icon="compass"></b-icon>
                            <span>現在地</span>
                          </button>
                        <div id="map"></div>
                    </section>
                    <footer class="modal-card-foot">
                        <button class="button" type="button" @click="$parent.close()">Close</button>
                    </footer>
                </div>
</template>

<script>
const loadGoogleMapsApi = require('load-google-maps-api')
export default {
  name: 'ModalForm',
  props: {
  },
    data:function(){
        return {
            hasGeolocation:("geolocation" in navigator),
            mapObj:{
                mapElm:{},
                latlng:{
                    //本州デフォルトセッティング
                    lat:37.8629704,
                    lng:136.4834582
                },
                zoom:6,
                //api
                autocomplete:{},
                geocoder:{},
                map:{}
            },
            addressBox:{
                value:'',
                inFocus:false
            },
            latlongBox:{
                value:'',
                inFocus:false,
                correct:true
            },
            googleMaps:{},
            targetMapElem:{},
        }
    },
    computed:{
        latLongBoxStatuses:function(){
            let msg = "緯度経度情報は[緯度 (latitude) : -90 〜 90,経度 (longitude) : -180 〜 180]による記入のみです。";
            let msg2 = "情報の反映は入力反映ボタンを押してください。"
            let msgline = "";
            let labelType = '';
            if(!this.latlongBox.correct){
                msgline += msg += this.latlongBox.inFocus?'<br>':'';
                labelType = 'is-danger';
            }
            if(this.latlongBox.inFocus){
                msgline += msg2;
            }
            let obj = {
                labelMessage:msgline,
                labelType
            }
            return obj
        }
    },
    watch:{
        'mapObj.latlng':{
            handler: function (val, oldVal) {
              //console.log(val)
             this.latlongBox.value = this.makeLatlongLine();
                const map = this.mapObj.map;
                if(map.center){
                    if(map.getCenter().lat() != val.lat || map.getCenter().lng() != val.ong){
                        map.setCenter(val);
                    }
                }
            },
            deep: true
        }
    },
    methods:{
        initGoogleMaps:function(){
            const self = this;
            loadGoogleMapsApi({
                key: 'AIzaSyApvOihEARIgFwXudP7i6IM2oob9QryW7g',
                language:'ja',
                libraries:['places']
            }).then(function (googleMaps) {
                self.googleMaps = googleMaps;
                self.initMap();
                self.initGeocorder();
                //self.initAutocomplete();
            }).catch(function (error) {
              console.error(error)
            })
            //初期データの設定
        },
        initMap:function(){
            const self = this;
              this.mapObj.map =  new this.googleMaps.Map(this.mapObj.mapElm, {
            center: this.mapObj.latlng,
            zoom: this.mapObj.zoom,
                  fullscreenControl:false,
                  mapTypeControl:false,
                  streetViewControl:false
          });
            this.mapObj.map.addListener("click", self.clickMap(e));
        },
        clickMap:function(e){
            console.log(e);
        },
        initGeocorder:function(){
            this.mapObj.geocoder = new this.googleMaps.Geocoder();
        },
        initAutocomplete:function(){
                //対応させるテキストボックス
                let input = document.querySelector('.addresses .input');
                //プレイスを検索する領域
                let LatLngFrom = this.googleMaps.LatLng(35.692195,139.7576653);
                let LatLngTo   = this.googleMaps.LatLng(35.696157,139.7525771);
                let bounds = this.googleMaps.LatLngBounds(LatLngFrom, LatLngTo);
              //オートコンプリートのオプション
                let options = {
                    types: ['geocode'],                      // 検索タイプ
                    bounds: bounds,                            // 範囲優先検索
                    componentRestrictions: {country: 'jp'}     // 日本国内の住所のみ
                };
                this.mapObj.autocomplete = new this.googleMaps.places.Autocomplete( input, options);
        },
        //UIctrl
        UIctrl:function(){
            const self = this;
            return {
              toast:function(Msg,Mode,Position,Duration){
            /*is-dark, is-white, is-black, is-light, is-dark, is-primary, is-info, is-success, is-warning, is-danger*/
                let type = Mode||'is-white';
                let msg = Msg||'';
                  /*is-top-right, is-top, is-top-left, is-bottom-right, is-bottom, is-bottom-left*/
                let position = Position||'is-top';
                let duration = Duration||5000;
                self.$toast.open({
                    duration: duration,
                    message: msg,
                    position: position,
                    type: type
                })
                }
            }

        },
        doForcus:function(targetmember,b,ev){
              targetmember.inFocus = b;
        },
        checkLatLngValidator:function(textline){
            let latlngArray = textline.split(',');
            
            const reg=/-?\d+\.?\d*[,]-?\d+\.?\d*/;
            let mutched = reg.exec(textline);
            if(!mutched||mutched[0]!=mutched.input){
                this.latlongBox.correct = false;
            }else{
                this.latlongBox.correct = true;
            }
                return mutched;
        },
        makeLatlongLine:function(){
            let geoline = `${this.mapObj.latlng.lat},${this.mapObj.latlng.lng}`;
            if(geoline.length<3){
                return '';
            }
            return geoline;
        },
        getGeocodeFromAddress:function(keyword){
            const self = this;
            this.mapObj.geocoder.geocode({
                  address: keyword
  }, function(results, status) {
    if (status == self.googleMaps.GeocoderStatus.OK) {

      // 結果の表示範囲。結果が１つとは限らないので、LatLngBoundsで用意。
      let bounds = new self.googleMaps.LatLngBounds();
      let map = self.mapObj.map;
      for (let i in results) {
        if (results[i].geometry) {

          // 緯度経度を取得
          let latlng = results[i].geometry.location;

          // 住所を取得(日本の場合だけ「日本, 」を削除)
          let address = results[0].formatted_address.replace(/^日本, /, '');

          // 検索結果地が含まれるように範囲を拡大
          bounds.extend(latlng);

          // あとはご自由に・・・。
          new google.maps.InfoWindow({
            content: address + "<br>(Lat, Lng) = " + latlng.toString()
          }).open(map, new google.maps.Marker({
            position: latlng,
            map: map
          }));
        }
      }

      // 範囲を移動
      map.fitBounds(bounds);

    } else if (status == google.maps.GeocoderStatus.ERROR) {
      alert("サーバとの通信時に何らかのエラーが発生！");
    } else if (status == google.maps.GeocoderStatus.INVALID_REQUEST) {
      alert("リクエストに問題アリ！geocode()に渡すGeocoderRequestを確認せよ！！");
    } else if (status == google.maps.GeocoderStatus.OVER_QUERY_LIMIT) {
      alert("短時間にクエリを送りすぎ！落ち着いて！！");
    } else if (status == google.maps.GeocoderStatus.REQUEST_DENIED) {
      alert("このページではジオコーダの利用が許可されていない！・・・なぜ！？");
    } else if (status == google.maps.GeocoderStatus.UNKNOWN_ERROR) {
      alert("サーバ側でなんらかのトラブルが発生した模様。再挑戦されたし。");
    } else if (status == google.maps.GeocoderStatus.ZERO_RESULTS) {
      alert("見つかりません");
    } else {
      alert("えぇ～っと・・、バージョンアップ？");
    }
  });
        },
        placeSearch:function(){
            
        }
    },
    created: function(){
    this.latlongBox.value = this.makeLatlongLine();
    },
    mounted: function(){
        const self = this;
        if(this.hasGeolocation){
            navigator.geolocation.getCurrentPosition(function(position) {
                self.mapObj.latlng.lat = position.coords.latitude;
                self.mapObj.latlng.lng = position.coords.longitude;
                self.UIctrl().toast('現在地で初期化されました','is-info');
            },function(err){
            self.UIctrl().toast('GPS情報は取得できませんでした','is-info');
            });
        }else{
            this.UIctrl().toast('この端末でのGPS情報は取得できません','is-warning');
        }
        document.querySelector('.modal .animation-content').style.maxWidth = '80vw';
        this.mapObj.mapElm = document.getElementById('map');
        this.initGoogleMaps();
    }
    
}
</script>
<style lang="scss">
    .modal-card{
            max-width: 80vw !important;
            min-width: 80vw !important;
            width: 80vw;
        }
      #map{
        height: 50vh;
    }
    .latlng .input[disabled]{
        background-color: white;
        border-color: #dbdbdb;
        box-shadow: inset 0 1px 2px rgba(10, 10, 10, 0.1);
    }
</style>