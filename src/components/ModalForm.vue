<template>
                <div class="modal-card">
                    <section class="modal-card-body">
                        
                        <div id="mapwrapper">
                        <div id="map"></div></div>
                        <div id="addressSearchBase">
                        <b-field label="住所検索" class="field_base">
                            <form class="addressSearch_wrapper" @submit.prevent="getGeocode({address:searchBox.value},true)">                                
                            <b-input
                                placeholder="住所キーワードから検索できます" icon="magnify" class="" onfocus="this.select();" v-model.lazy="searchBox.value">
                            </b-input>
                            <button class="button is-primary" type="submit">検索</button>
                            </form>
                                <transition name="result-trans">
                            <div class="addressSearchResult_base" v-show="transFlag">
                                    
                                <div class="addressLine"><b-icon icon="map-search-outline" size = "is-small" style="color:#666666"></b-icon>{{addressBox.value}}</div>
                                <div class="latlngLine"><b-icon icon="earth" size = "is-small"></b-icon>{{latlongBox.value}}</div>
    
                            </div>
                                </transition>
                        </b-field>
    
    </div>
                        
                          <button class="button getNowPosition" :disabled="!hasGeolocation" @click="rewindPosition()" style="color:#666666">
                            <b-icon icon="crosshairs-gps"></b-icon>
                          </button>
                    </section>
                    <footer class="modal-card-foot">
                        <button class="button" type="button" @click="$parent.close()">キャンセル</button>
                        <button class="button" type="button" @click="setPlaceData">情報設定</button>
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
            transFlag:true,
            hasGeolocation:("geolocation" in navigator),
            isGPSworking:false,
            mapObj:{
                mapElm:{},
                latLng:{
                    //本州デフォルトセッティング
                    lat:37.8629704,
                    lng:136.4834582
                },
                zoom:6,
                marker:{},
                infoWindow:{},
                //api
                autocomplete:{},
                geocoder:{},
                map:{}
            },
            searchBox:{
                value:''
            },
            addressBox:{
                value:''
            },
            latlongBox:{
                value:''
            },
            target:{
                address:{},
                latlng:{}
            },
            googleMaps:{},
            targetMapElem:{},
        }
    },
    computed:{
    },
    watch:{
        'mapObj.latLng':{
            handler: function (val, oldVal) {
              //console.log(val)
             this.latlongBox.value = this.mapObj.latLng.toString();
                /*const map = this.mapObj.map;
                if(map.center){
                    if(map.getCenter().lat() != val.lat || map.getCenter().lng() != val.ong){
                        map.setCenter(val);
                    }
                }*/
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
                self.initDefaultLatLng();
                self.initMap();
                self.initGeocorder();
                self.setNowPosition(function(b){
                    if(b){
                        self.setMarkerFirst();
                    }
                });
                //self.initAutocomplete();
            }).catch(function (error) {
              console.error(error)
            })
            //初期データの設定
        },
        initMap:function(){
            const self = this;
              this.mapObj.map =  new this.googleMaps.Map(this.mapObj.mapElm, {
            center: this.mapObj.latLng,
            zoom: this.mapObj.zoom,
                  fullscreenControl:false,
                  mapTypeControl:false,
                  streetViewControl:false
          });
            this.mapObj.map.addListener("click", self.clickMap);
        },
        initDefaultLatLng:function(){
            this.mapObj.latLng = new google.maps.LatLng(this.mapObj.latLng)
        },
        clickMap:function(ev){
          const map = this.mapObj.map;
            const marker = this.mapObj.marker;
          const bd = map.getBounds();
            const touchLatLng = ev.latLng;
            const pinLatLng = marker.getPosition();
            console.log(bd.intersects(new google.maps.LatLngBounds(pinLatLng)));
            if(!bd.intersects(new google.maps.LatLngBounds(pinLatLng))){
            marker.setPosition(ev.latLng);
            this.dragendMarker(ev);
            }
        },
        setMarkerFirst:function(){
            this.mapObj.marker = new google.maps.Marker({
              position: this.mapObj.latLng,
              map: this.mapObj.map,
              draggable: true,
              animation: google.maps.Animation.DROP
            });
            
            this.mapObj.infoWindow = new google.maps.InfoWindow({content:''});
            this.getGeocode({location:this.mapObj.latLng},true);
            
            let map = this.mapObj.map;
            let marker = this.mapObj.marker;
            let infoWindow = this.mapObj.infoWindow;
            
		   marker.addListener("dragstart", function() {
			infoWindow.close();
		});
            marker.addListener("dragend",this.dragendMarker);
            marker.addListener("click",function(){
                infoWindow.open(map,marker);
            })
        },
        dragendMarker:function(ev){
            const map = this.mapObj.map;
            const marker = this.mapObj.marker;
            const infoWindow = this.mapObj.infoWindow;
            const self = this;
                let changeButton = document.createElement("button");
                changeButton.innerText = "再取得";
                changeButton.addEventListener("click",function(){
                    console.log("geocode research");
                    self.getGeocode({location:ev.latLng},false);
                },false);
                let contents = document.createElement('div');
                contents.innerHTML = "マーカー位置が変更されました。この位置情報で再取得しますか？<br>(lat,lng) = "+ev.latLng.toString()+"<br>";
                contents.appendChild(changeButton);
                infoWindow.setContent(contents);
                infoWindow.open(map, marker);
        },
        rewindPosition:function(){
            const map = this.mapObj.map;
            const marker = this.mapObj.marker;
            const self = this;
            this.setNowPosition(function(result){
                if(result){
                    self.getGeocode({location:self.mapObj.latLng},true);
                }
            })
        },
        setNowPosition:function(callBack){
            const self = this;
            const callback = callBack;
            if(this.hasGeolocation){
                navigator.geolocation.getCurrentPosition(function(position) {
                    self.isGPSworking = true;
                    self.mapObj.zoom = 16;
                    self.mapObj.latLng = new self.googleMaps.LatLng({lat:position.coords.latitude,lng:position.coords.longitude}) ;
                    self.UIctrl().toast('現在地で初期化されました','is-info');
                    if(callback)callback(true);
                        //self.initGoogleMaps();
                },function(err){
                self.UIctrl().toast('GPS情報は取得できませんでした','is-info');
                    
                    if(callback)callback(false);
                });
            }else{
                this.UIctrl().toast('この端末でのGPS情報は取得できません','is-warning');
                    if(callback)callback(false);
            }
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
        getGeocode:function(keywordObj,dofitBounds){
            this.searchBox.value="";
            this.transFlag=false;
            const doFitBounds = dofitBounds;
            const self = this;
            const isReverse = (keywordObj.location)?true:false;
            const position = isReverse?keywordObj.location:{};
            //if(isReverse){
              //  keywordObj.location_type = 'APPROXIMATE';
            //}
            this.mapObj.geocoder.geocode(keywordObj, function(results, status) {
                    if (status == self.googleMaps.GeocoderStatus.OK) {

                      // 結果の表示範囲。結果が１つとは限らないので、LatLngBoundsで用意。
                      let bounds = new self.googleMaps.LatLngBounds();
                      let map = self.mapObj.map;
                      let infoWindow = self.mapObj.infoWindow;
                        let marker = self.mapObj.marker;
                        let latlng = self.mapObj.latLng;
                        
                        let formatedResults = [];
                        
                        for(let i=0;i<results.length;i++){
                            if(!isReverse){
                            formatedResults=results;
                                break;
                            }else{
                                if(results[i].geometry.location_type == "APPROXIMATE"){
                                    for(let val of results[i].types){
                                        if(val == "political"){
                                            formatedResults.push(results[i]);
                                            break;
                                        }
                                    }
                                }
                            }
                        }

                        if (formatedResults[0].geometry) {

                          // 緯度経度を取得
                            //let location = !isReverse?formatedResults[0].geometry.location:{lat:function(){
                              //  return 
                            //}}
                          let latlng = isReverse?position:formatedResults[0].geometry.location;

                          // 住所を取得(日本の場合だけ「日本, 」を削除)
                          let address = formatedResults[0].formatted_address.replace(/^日本、/, '');
                            
                            self.mapObj.latLng = latlng;
                            
                            self.addressBox.value = address;

                          // 検索結果地が含まれるように範囲を拡大
                          //bounds.extend(results[0]);
                          /*new google.maps.InfoWindow({
                            content: address + "<br>(Lat, Lng) = " + latlng.toString()
                          }).open(map, self.mapObj.marker);*/
                            marker.setPosition(latlng);
                            infoWindow.setContent(address + "<br>(Lat, Lng) = " + latlng.toString());
                            infoWindow.open(map, marker);
                        }
                      // 範囲を移動
                      if(doFitBounds){
                          if(formatedResults[0].geometry.bounds){
                          map.fitBounds(formatedResults[0].geometry.bounds);
                          }
                      }

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
                      alert("不明のエラー");
                    }
                setTimeout(function(){
                    self.transFlag=true;
                },1000);
                  }
                                        );
        },
        setPlaceData:function(){
            this.target.address.value = this.addressBox.value;
            this.target.latlng.value = this.latlongBox.value.match(/\((.*)\)/i)[1];
            this.$parent.close();
        }
    },
    created: function(){
        this.target.address = document.querySelector('#addAddress');
        this.target.latlng = document.querySelector('#addLatlng');
    },
    mounted: function(){
        this.initGoogleMaps();
        //document.querySelector('.modal .animation-content').style.maxWidth = '90vw';
        this.mapObj.mapElm = document.getElementById('map');
    }
    
}
</script>
<style lang="scss">
    .modal-card-body{
        padding: 0 !important;
            overflow: hidden;
    }
    .modal-card{
            max-height: calc(100vh - 3vw) !important;
            margin: 0 auto !important;
            //max-width: 97vw !important;
            //min-width: 97vw !important;
            //width: 97vw;
        border-radius: 1em;
        }
    #mapwrapper{
        width: 100%;
        padding-bottom: 75px;
        overflow: hidden;
        border-radius: 1em;
    }
      #map{
    width: 100%;
    height: -webkit-fill-available;
          height: -moz-available;
          height: fill;
          height: available;
    }
    .getNowPosition{
        right: 8px;
        position: absolute !important;
        bottom: 200px;
        width: 42px;
        height: 42px !important;
    }
    #addressSearchBase{
    position: absolute;
    top: 1vw;
    left: 50%;
    width: 98%;
    transform: translateX(-50%);
        .label{
            font-size: 0.8rem;
        }
        .help {
            font-size: 0.8rem;
            margin-top: 0.25rem;
        }
        .field.has-addons{
            flex-direction: column;
        }
        .addressSearchResult_base{
            font-size: 0.85rem;
            line-height: 178%;
        }
    }
    .field_base{
        background-color: rgba(255, 255, 255, 0.8);
        padding: 1vw;
        border-radius: 1em;
        border: 1px solid rgba(0,0,0,0.3);
    }
    .addressSearch_wrapper{
        display: flex;
        .control{
            width: 100%;
        }
    }
    .modal .animation-content{
        margin: 0 3vw !important;
    }
    .latlng .input[disabled]{
        background-color: white;
        color: blue;
        border-color: #dbdbdb;
        box-shadow: inset 0 1px 2px rgba(10, 10, 10, 0.1);
    }
    .result-trans-enter-active{
        transition:all 1s ease;
    }
    .result-trans-leave-active{
        transition: all .5s ease;
    }
    .result-trans-enter, .result-trans-leave-to{
        opacity: 0;
        transform: translateX(15px)
    }
    
</style>