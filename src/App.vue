<template>
  <div id="app">
    <img src="./assets/img.png" alt="btc" class="btc">
    <HelloWorld/>
  </div>
</template>

<script>
import HelloWorld from "@/components/HelloWorld";


export default {
  name: 'App',
  components: {
    HelloWorld,

  }
}
</script>

<style scoped>
  #app{
    box-sizing: border-box;
    margin: 0;
  }
.btc{
  position: absolute;
  width: 60px;
  left: 50%;
  top: 30px;
  animation: rotation 10s infinite;
}
@media (max-width: 600px) {
  .btc{
    width: 30px;
  }
}
  @keyframes rotation {
    from{
      transform: rotateX(0deg);

    }
    to{
      transform: rotateX(360deg);

    }
  }
</style>
