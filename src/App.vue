<template>
    <section>
        
                            <b-input
                                placeholder="現場住所入力" id="addAddress" @focus="modalOpen">
                            </b-input>
        
                            <b-input
                                placeholder="経度緯度入力" id="addLatlng" @focus="modalOpen">
                            </b-input>

        <b-modal :active.sync="isComponentModalActive" has-modal-card>
            <modal-form ></modal-form>
        </b-modal>
    </section>
</template>

<script>
    
import ModalForm from './components/ModalForm.vue'
    export default {
        components: {
            ModalForm
        },
        data() {
            return {
                isComponentModalActive: false
            }
        },
        methods:{
            modalOpen(){
                this.$modal.open({
                    parent:this,
                    component:ModalForm,
                    hasModalCard:true,
                    canCancel:['escape']
                })
            }
        }
    }
</script>
<style>
      html, body {
        height: 100%;
        margin: 0;
        padding: 0;
      }
</style>