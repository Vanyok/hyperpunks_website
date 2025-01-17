<template>
  <v-app dark>
    <v-app-bar
      v-if="!countdownFinished"
      class="app-nav"
      color="transparent"
      clipped-left
      app
      elevation="0"
    >
      <v-toolbar-title>
        <nuxt-link style="text-decoration: none" to="/">
          <img
            src="/logo.png"
            class="logo-img"
            style="height: 60px"
            alt="hyperpunks logo"
          />
        </nuxt-link>
      </v-toolbar-title>
      <div class="flex-grow-1"></div>
      <div class="top-links">
        <nuxt-link to="/" class="margin-right-20"> SEARCH </nuxt-link>
        <nuxt-link to="/list" class="margin-right-20"> LIST </nuxt-link>
      </div>
      <v-btn
        href="https://twitter.com/HyperPunks_NFT"
        target="_blank"
        fab
        text
        redtext
      >
        <v-icon>mdi-twitter</v-icon>
      </v-btn>

      <v-btn
        text
        fab
        @click="goToExternalUrl('https://medium.com/@hyperpunks')"
      >
        <img src="/medium.svg" width="30" alt="hyperpunks blog" />
      </v-btn>

      <v-btn class="ma-2" text @click="metamaskButtonClicked()">{{
        walletBtnText
      }}</v-btn>
    </v-app-bar>

    <v-main>
      <nuxt />
    </v-main>
    <v-footer v-if="showNonMainnetWarning" color="transparent" app>
      <v-card class="flex" flat>
        <v-card-text class="red white--text text-center">
          <strong>Warning!</strong> Not connected to Ethereum Mainnet (network
          ID: {{ $store.state.networkID }})
        </v-card-text>
      </v-card>
    </v-footer>
  </v-app>
</template>
<script>
import { ethers } from 'ethers'
import { DEADLINE, RPC_PROVIDER, NETWORK_ID } from '../constants'
export default {
  auth: false,
  data() {
    return {
      walletBtnText: 'CONNECT WALLET',
      ethers: null,
      provider: null,
      showNonMainnetWarning: false,
      countdownFinished: false,
    }
  },
  mounted() {
    const t = Date.parse(DEADLINE) - Date.parse(new Date())
    if (t > 0) {
      this.countdownFinished = true
      this.$router.push('/countdown')
      return
    }
    this.rpcProviderInit()
  },
  methods: {
    async rpcProviderInit() {
      if (!window.ethereum) {
        this.provider = 'not_web3'
        this.ethers = new ethers.providers.JsonRpcProvider(
          RPC_PROVIDER,
          NETWORK_ID
        )
        console.log('this.provider :>> ', this.provider)
        return
      }

      await window.ethereum.enable()
      this.provider = 'web3'
      console.log('this.provider :>> ', this.provider)
      this.ethers = new ethers.providers.Web3Provider(window.ethereum, 'any')
      this.ethers.on('network', (newNetwork, oldNetwork) => {
        if (oldNetwork) {
          window.location.reload()
        }
      })

      this.signer = this.ethers.getSigner()
      this.account = await this.signer.getAddress()
      this.balance = await this.signer.getBalance()
      this.ethBalance = await ethers.utils.formatEther(this.balance)
      this.signer = this.ethers.getSigner()
      const addr = await this.signer.getAddress()
      this.walletBtnText =
        addr.substr(0, 7) + '...' + addr.substr(addr.length - 5, addr.length)

      window.ethereum.on('accountsChanged', (accounts) => {
        if (accounts.length < 1) return

        this.walletBtnText =
          accounts[0].substr(0, 7) +
          '...' +
          accounts[0].substr(accounts[0].length - 5, accounts[0].length)
      })

      const { chainId } = await this.ethers.getNetwork()
      if (chainId !== 1) {
        this.showNonMainnetWarning = true
      }
    },
    goToUrl(url) {
      this.$router.push(url)
    },
    goToExternalUrl(url) {
      window.open(url, '_blank')
    },
    metamaskButtonClicked() {
      if (window.ethereum) {
        this.rpcProviderInit()
      } else {
        this.$router.push('/other/install_metamask')
      }
    },
  },
}
</script>
<style scoped>
#styled-input {
  height: 48px;
  width: 48px;
}
.styled-input label[for] {
  height: 48px;
}
.v-navigation-drawer > .list:not(.list--dense) .list__tile {
  font-size: 17px;
}
.margin-right-20 {
  margin-right: 20px;
}
.avatar {
  max-width: 75px;
}
/* .list__tile--active.list__tile.list__tile--link {
} */
a.nuxt-link-exact-active.list__tile--active.list__tile.list__tile--link {
  font-weight: 900 !important;
}
.v-list-item {
  border-left: 10px solid transparent;
}
.v-list-item--active {
  color: #333;
  border-left: 10px solid #ff5722;
}
@media screen and (max-width: 756px) {
  .top-links {
    display: flex;
    width: 100%;
    justify-content: center;
  }
}
</style>
