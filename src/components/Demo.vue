<template>
  <div class="hello" style="padding-top: 20px">
    <h1>这里是Web3.js的一个演示组件</h1>
    <div class="content">
      以下几个案例将分别介绍使用web3.js组件连接用户钱包,切换网络,读取链上内容,写入链上内容的简单流程<br />
      你可以在这里获取web3.js的中文文档:&nbsp;<a
        href="https://learnblockchain.cn/docs/web3.js/"
        >登链社区</a
      >&nbsp;&nbsp;<a href="https://www.qikegu.com/docs/5127">奇客谷</a>
    </div>

    <h1>Step1 连接钱包</h1>
    <div class="content">
      如果你是初学者,你需要在你的浏览器(本例为Chrome)安装Metamask(也称小狐狸钱包),点击安装<br /><br /><a
        href="https://metamask.io/"
        target="_blank"
        ><img src="../assets/mm-logo.svg" alt=""
      /></a>
    </div>
    <div class="content">当你安装好且生成了一个新钱包后</div>
    <button @click="connect_wallet" class="btn">点击连接钱包</button>
    <div class="content">
      当前钱包地址:<span>{{ address ? address : "用户未连接" }}</span
      ><br />
      当前链ID:{{ chainId }}<br />目标链ID(config.js中配置):{{ targetChainId }}
    </div>
    <button @click="change_chain" class="btn" v-if="targetChainId != chainId">
      切换链
    </button>

    <h1>区块链读操作</h1>
    <div class="content">
      当前测试智能合约在Rinkeby链上,已开源
      <a
        href="https://rinkeby.etherscan.io/address/0x447d2F2F3C7cfB946BAe1DD28B4CaD3Bcf5d1Ec5#readContract"
        >合约地址</a
      >
    </div>
    <div class="content">
      当前市场流通总量:&nbsp;<b>{{ totalSupply }} </b>&nbsp;&nbsp;<button
        @click="getSupply()"
      >
        点击获取
      </button>
    </div>
    <div class="content">
      当前钱包地址持有NFT数量 <b>{{ balance }}</b> 当前用户是否是被邀请用户:
      <b> {{ hive }} </b> <button @click="getBalance()">点击判断</button>
    </div>
    <h1>区块链写操作</h1>
    <div class="content">
      点击下面按钮,将会给你当前账户免费Mint一个NFT,你可以在<a :href="`https://rinkeby.etherscan.io/address/${address}`">这里</a>查看你的地址在链上的信息<br>
      点击Mint一个☝🏻NFT给当前账户
      <button @click="mint()">点击Mint</button>
    </div>
    <div class="content">{{ mint_state }}</div>
    <div class="content">这个过程可能需要测试币(ETH),你可以在这里获取<a href="https://faucet.rinkeby.io/">Rinkeby水龙头</a></div>
  </div>
</template>

<script>
import abi from "@/abi.json";
import Web3 from "web3";
import config from "@/config";
export default {
  name: "HelloWorld",
  data() {
    return {
      web3: null,
      address: null,
      chainId: null,
      targetChainId: null,
      totalSupply: "未获取",
      hive: "未获取",
      contract: null,
      balance: "未查询",
      mint_state: "未开始",
    };
  },
  methods: {
    mint() {
      let self = this;
      self.mint_state = "进行中...";
      self.contract.methods
        .mint().send({
          from: self.address,
        }).then((e) => {
          console.log(e);
          self.mint_state = `完成,交易哈希:${e.blockHash}`;
          self.getBalance();
        })
        .catch((e) => {
          console.log('发生错误',e);
        });
    },
    async getBalance() {
      let self = this;
      self.hive = "查询中...";
      self.balance = "查询中...";
      let amount = await self.contract.methods.balanceOf(self.address).call();
      self.balance = amount;
      self.hive = amount > 0;
    },
    async getSupply() {
      let self = this;
      self.totalSupply = "获取中...";
      let amount = await self.contract.methods.totalSupply().call();
      self.totalSupply = amount;
    },
    async change_chain() {
      let self = this;
      window.ethereum
        .request({
          method: "wallet_switchEthereumChain",
          params: [{ chainId: config.chainId }],
        })
        .then((res) => {
          // 当切换链成功后,刷新页面
          window.location.reload();
        })
        .catch((error) => {});
    },
    async connect_wallet() {
      let self = this;
      let web3Provider;
      if (window.ethereum) {
        web3Provider = window.ethereum;
        try {
          // 请求用户授权
          await window.ethereum.enable();
        } catch (error) {
          // 此处用户不给授权的处理逻辑
        }
      } else if (window.web3) {
        //windows.web3是用来适配旧版metamask
        web3Provider = window.web3.currentProvider;
      } else {
        // 处理用户没有metamask的逻辑
      }

      web3 = new Web3(web3Provider);
      self.web3 = web3;
      // 获取用户钱包地址列表,通常一个metamask钱包考虑性能最多可生成30多个地址
      let accounts = await self.web3.eth.getAccounts();
      let chainId = await self.web3.eth.getChainId();

      self.chainId = chainId;
      if (accounts.length) {
        self.address = accounts[0];
      }
      window.ethereum.on("chainChanged", (chainId) => {
        console.log("用户切换了链");
      });
    },
  },
  async mounted() {
    let self = this;
    self.targetChainId = parseInt(config.chainId);
    // 每当用户打开网页,则要求他链接钱包
    await self.connect_wallet();
    // 绑定钱包后,绑定目标智能合约的实例
    self.contract = new self.web3.eth.Contract(abi, config.contractAddress);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.hello {
  .content {
    font-size: 23px;
    line-height: 40px;
    margin-bottom: 20px;
  }
  .btn {
    height: 60px;
    padding: 0px 15px;
    font-size: 23px;
    line-height: 60px;
    margin-bottom: 20px;
  }
}
</style>
