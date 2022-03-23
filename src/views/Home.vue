<template>
  <div class="home">
    <!-- 可以切换ethers和web3.js 的demo组件 -->
    <!-- 首推使用ethers.js -->
    <ethers-demo></ethers-demo>
    <!-- <web3-demo></web3-demo> -->
  </div>
</template>

<script>
// @ is an alias to /src
import Web3Demo from '@/components/Web3JsDemo.vue'
import EthersDemo from '@/components/EthersDemo.vue'

export default {
  name: 'Home',
  components: {
      Web3Demo,EthersDemo
  },
  
}
</script>
