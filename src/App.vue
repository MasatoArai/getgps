<template>
  <div id="app">
    
    <section>
        <button class="button is-medium is-dark" @click="prompt">
            Launch prompt (default)
        </button>

        <button class="button is-medium is-dark" @click="promptNumber">
            Launch prompt (number)
        </button>
    </section>
      
    <button type="button"  class="button is-medium is-dark" @click="open">Open Modal</button>
    <modal name="example">
      <div class="basic-modal">
        <iframe id="mm" src=""></iframe>
        <div id="show_result"></div>
        <button class="button" type="button" @click="close">Close Modal</button>
      </div>
    </modal>
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'

    
export default {
  name: 'app',
  components: {
   // HelloWorld
  },
    methods: {
        prompt() {
            navigator.geolocation.getCurrentPosition(
successCallback, errorCallback
);
            function successCallback(position) {
  var gl_text = "緯度：" + position.coords.latitude + "<br>";
    gl_text += "経度：" + position.coords.longitude + "<br>";
    gl_text += "高度：" + position.coords.altitude + "<br>";
    gl_text += "緯度・経度の誤差：" + position.coords.accuracy + "<br>";
    gl_text += "高度の誤差：" + position.coords.altitudeAccuracy + "<br>";
    gl_text += "方角：" + position.coords.heading + "<br>";
    gl_text += "速度：" + position.coords.speed + "<br>";
  document.getElementById("show_result").innerHTML = gl_text;
}
            function errorCallback(e){
                console.log(e);
            }
            this.$dialog.prompt({
                message: `What's your name?`,
                inputAttrs: {
                    placeholder: 'e.g. Walter',
                    maxlength: 10
                },
                onConfirm: (value) => this.$toast.open(`Your name is: ${value}`)
            })
        },
        promptNumber() {
            this.$dialog.prompt({
                message: `What's your age?`,
                inputAttrs: {
                    type: 'number',
                    placeholder: 'Type your age',
                    value: '18',
                    maxlength: 2,
                    min: 18
                },
                onConfirm: (value) => this.$toast.open(`Your age is: ${value}`)
            })
        },
        
        open () {
          this.$modal.push('example');
            navigator.geolocation.getCurrentPosition(successCallback, errorCallback);
            function successCallback(position) {
              var gl_text = "緯度：" + position.coords.latitude + "<br>";
                gl_text += "経度：" + position.coords.longitude;
              document.getElementById("show_result").innerHTML = gl_text;
            let dl = document.getElementById('mm').src="https://mapfan.com/map?c="+position.coords.latitude+","+position.coords.longitude+",15";
            console.log(dl);
            }
                function errorCallback(e){
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
        width: 100%;
        height: 80vw;
        border: 1px solid black;
    }
</style>
