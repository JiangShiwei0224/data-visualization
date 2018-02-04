<template>
  <div class="box">
      <input v-show="show" name="word" HaoyuSug="45B23A8781D64E62A77691E41BDB2C16" @blur="show=!show,value =''" v-model="value"  type="text" placeholder="请输入变量名..." class="textInput">    
      <input @click="show = !show" v-if="!show" type="submit" value="     " class="inputButton">            
  </div>
</template>

<script>
    export default {
        name: 'searchInput',
        data () {
            return {
                show: false,
                value: ''
            }
        },
        computed: {
        },
        mounted(){
        },
        methods: {
        }
    }
</script>

<style scoped>
.container{
    position: relative;
}
    .textInput{
        width: 250px;
        height: 20px;
        position: absolute;
        right: 20px;
    }
    .inputButton{
        height: 25px;
        z-index: 10;
        position: absolute;
        right: 20px;
        background-image: url(../assets/mirror.png);
        background-size: 180% 150%;
        background-repeat: no-repeat;
        border: none;
        outline: none;
    }
</style>

