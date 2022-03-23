<template>
  <div class="hello" style="padding-top: 20px">
    <h1>这里是ethers.js的一个演示组件</h1>
    <div class="content">
      以下几个案例将分别介绍使用ethers.js组件连接用户钱包,切换网络,读取链上内容,写入链上内容的简单流程<br />
      你可以在这里获取ethers.js的中文文档:&nbsp;<a
        href="https://learnblockchain.cn/docs/ethers.js/getting-started.html"
        >登链社区</a
      >
    </div>

    <h1>Step1 连接钱包(connect_wallet 函数)</h1>
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
      下面的例子是NFT的一个实用功能,通过判断用户账户是否有NFT来判断用户是不是被邀请的客人(NFT作为邀请函使用)
    </div>
    <div class="content">
      当前钱包地址持有NFT数量 <b>{{ balance }}</b> 当前用户是否是被邀请用户:
      <b> {{ isGuests }} </b> <button @click="getBalance()">点击判断</button>
    </div>
    <h1>区块链写操作</h1>
    <div class="content">
      点击下面按钮,将会给你当前账户免费Mint一个NFT,你可以在<a
        :href="`https://rinkeby.etherscan.io/address/${address}`"
        >这里</a
      >查看你的地址在链上的信息<br />
      点击Mint一个☝🏻NFT给当前账户
      <button @click="mint()">点击Mint</button>
    </div>
    <div class="content">{{ mint_state }}</div>
    <div class="content">
      这个过程可能需要测试币(ETH),你可以在这里获取<a
        href="https://faucet.rinkeby.io/"
        >Rinkeby水龙头</a
      >
    </div>
  </div>
</template>

<script>
import abi from "@/abi.json";
import { ethers } from "ethers";
import config from "@/config";
export default {
  name: "EthersDemo",
  data() {
    return {
      web3: null,
      address: null,
      chainId: null,
      targetChainId: null,
      totalSupply: "未获取",
      isGuests: "未获取",
      contract: null,
      balance: "未查询",
      mint_state: "未开始",
    };
  },
  methods: {
    async mint() {
        //这个函数是一个智能合约写函数,用来和合约交互,并获得一个nft
        //与读函数不同,此处contract实例需要有一个connect signer的操作
      let self = this;
      let signer = self.web3.getSigner();
      const contract = self.contract.connect(signer);
      self.mint_state = "进行中...";
       contract.mint().then((e) => {
          self.mint_state = `完成,交易哈希:${e.blockHash}`;
          self.getBalance();
        })
        .catch((e) => {
          console.log("发生错误", e);
        });
    },
    async getBalance() {
      // 这个函数是一个读取函数,用来获取当前用户地址拥有多少NFT
      //ethers的请求结构更简单,不需要methods,也不需要call
      let self = this;
      self.isGuests = "查询中...";
      self.balance = "查询中...";
      let amount = await self.contract.balanceOf(self.address);
    //下面是web3.js的结构
    //let amount = await self.contract.methods.balanceOf(self.address).call();
      self.balance = amount;
      self.isGuests = amount > 0;
    },
    async getSupply() {
      // 这个函数是一个读取函数,用来获取当前NFT合约当前的流通量(总共被人Mint了多少次)
      let self = this;
      self.totalSupply = "获取中...";
      let amount = await self.contract.totalSupply();
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
      //此函数是链接用户钱包功能,用来将ethers实例化
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
      web3 = new ethers.providers.Web3Provider(web3Provider);
      let user = web3.getSigner();
      //ether.js获取chainId等链信息比web3.js简洁且有逻辑的多,直接通过getNetWork即可获得链内容
      let networkInfo = await web3.getNetwork();
      // ethers.js和web3.js不同的点,在于可以直接获得当前钱包地址,通过上面的Signer(user)
      let wallet_address = await user.getAddress();
      let chainId = networkInfo.chainId;

      self.web3 = web3;
      self.chainId = chainId;
      self.address = wallet_address;
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
    //ethers的合约实例化传参,除了abi和地址,还需要传递一个provider,也就是刚才在connect_wallet里实例化的web3
    //abi和address顺序与web3.js相反
    //这个过程依赖self.web3,所以建议在connect_wallet结束的回调里使用,这里只做粗糙演示
    self.contract = new ethers.Contract(config.contractAddress,abi,self.web3);
    
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
