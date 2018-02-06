<template>
    <div class="container1">
        <div></div>
    </div>
</template>

<script>
export default {
    name: 'infoPrompt',
        data () {
            return {
                data: [
                    {
                        type: 'warn',
                        text: 'This is warn'
                    },
                    {
                        type: 'log',
                        text: 'This is log'
                    },
                    {
                        type: 'error',
                        text: 'This is error'
                    },
                    {
                        type: "info",
                        text: 'This is info'
                    }
                ]
            }
        },
        computed: {
        },
        mounted(){
            this.infoPromptInit(this.data);
        },
        methods: {
            infoPromptInit(data){
                var _this = this
                data.forEach(element => {
                    var oButton = document.createElement('button')
                    document.getElementsByTagName('div')[1].appendChild(oButton)
                    oButton.innerHTML = element.type;     
                    oButton.onclick = function (){
                        _this.infoPromptDisplay(element.text)
                    } 
                });
            },
            infoPromptDisplay(text){
                alert(text)
            }
        }
}
</script>

<style scoped>
    
</style>
