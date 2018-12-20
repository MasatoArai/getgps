<template>
                <div class="modal-card">
                    <header class="modal-card-head">
                        <p class="modal-card-title" style="text-align:center">位置情報設定</p>
                    </header>
                    <section class="modal-card-body">
                        <b-field label="住所検索" :message="addressBox.inFocus?'地域名をキーワードに住所を取得できます':''">
                            <div>
                            <b-input
                                placeholder="住所入力"
                                     icon="magnify" v-model="addressBox.value" @focus="doForcus(addressBox,true)" @blur="doForcus(addressBox,false)" >
                            </b-input>
                            <button class="button is-primary is-small">検索</button>
                            </div>
                        </b-field>

                        <b-field label="緯度経度情報"
                                 :type="latlongBox.correct?'':'is-danger'"
                                 :message="latLongBoxStatuses.labelMessage"
                                 >
                            <div>
                            <b-input
                                placeholder="緯度経度" @input="checkLatLngValidator" @focus="doForcus(latlongBox,true)" @blur="doForcus(latlongBox,false,$event)" v-model="latlongBox.value" icon="map-marker-radius">
                                
                            </b-input>
                            <button class="button is-primary is-small" :disabled="!latlongBox.correct">入力反映</button>
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
    
export default {
  name: 'ModalForm',
  props: {
  },
    data:function(){
        return {
            hasGeolocation:("geolocation" in navigator),
            latlong:{
                lat:'',
                lng:''
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
            map:{},
            targetMapElem:{},
            googleMapsClient:require('@google/maps').createClient({
              key: 'AIzaSyApvOihEARIgFwXudP7i6IM2oob9QryW7g'
            })
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
    methods:{
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
        initMap:function(){
              this.map = new google.maps.Map(this.targetMapElem, {
              center: {lat: -34.397, lng: 150.644},
              zoom: 8
            });
        },
        makeLatlongLine:function(){
            let geoline = `${this.latlong.lat},${this.latlong.lng}`;
            if(geoline.length<3){
                return '';
            }
            return geoline;
        },
        geocodeTest:function(keyword){
            this.googleMapsClient.geocode({
                  address: keyword
                }, function(err, response) {
                  if (!err) {
                    console.log(response.json.results);
                  }
                });
        }
    },
    created: function(){
    },
    mounted: function(){
        const self = this;
        if(this.hasGeolocation){
            navigator.geolocation.getCurrentPosition(function(position) {
                self.latlong.lat = position.coords.latitude;
                self.latlong.lng = position.coords.longitude;
                self.latlongBox.value = self.makeLatlongLine();
                self.UIctrl().toast('現在地で初期化されました','is-info');
            },function(err){
            self.UIctrl().toast('GPS情報は取得できませんでした','is-info');
            });
        }else{
            this.UIctrl().toast('この端末でのGPS情報は取得できません','is-warning');
        }
        
        this.targetMapElem = document.getElementById('map');
    }
    
}
</script>
<style lang="scss" scoped>
      #map{
        height: 100%;
    }
</style>