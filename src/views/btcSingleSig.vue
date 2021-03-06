<template>
  <v-layout align-center justify-center row fill-height>
    <v-flex xs11 >
      <v-card class="text-xs-center no-gutters" style="" v-if='!correctWalletExist'>
        <v-container>
          <v-row no-gutters justify='center' align='center'>
            <v-col
              cols="6"
              justify='center'
            >
            <v-alert
              dense
              type="error"
            >
              No Hardware Wallet Setup
            </v-alert>
            </v-col>
          </v-row>
          <v-row no-gutters justify='center' align='center'>
            <v-col
              cols="6"
              justify='center'
            >
            <v-alert
              dense
              type="info"
            >
              Please Setup Hardware Wallet
            </v-alert>
            </v-col>
          </v-row>
        </v-container>
      </v-card>
      <v-card class="text-xs-center no-gutters" style="" v-if='!walletReady && correctWalletExist'>
        <v-card-title class="headline justify-center">
          Wallet Syncing...
        </v-card-title>
            <v-divider/>
            <div class="text-center mb-5 mt-5">
              <v-progress-circular
                :size="100"
                :width="7"
                color="purple"
                indeterminate
              ></v-progress-circular>
            </div>
          <v-divider/>
          <v-card-actions>
          <v-btn
            color="orange"
            text
          >
            <v-icon>mdi-help</v-icon>
          </v-btn>
          <v-btn
            color="primary"
            text
          >
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
        </v-card-actions>
      </v-card>
      <v-card class="text-xs-center no-gutters" style=""
      v-if='!walletError && walletReady && correctWalletExist' :key='keyInfo'>
        <v-card-title class="headline justify-center">
          Bitcoin Wallet
        </v-card-title>
            <v-divider></v-divider>
            <v-tabs background-color="">
              <v-tab>
                <v-icon left>mdi-lock-clock</v-icon>
                Balance
              </v-tab>
              <v-tab>
                <v-icon left>mdi-email-receive</v-icon>
                Receive Money
              </v-tab>
              <v-tab>
                <v-icon left>mdi-send-check</v-icon>
                Send Money
              </v-tab>
              <v-tab>
                <v-icon left>mdi-history</v-icon>
                Transactions
              </v-tab>
              <v-tab-item>
                <balance/>
              </v-tab-item>
              <v-tab-item>
                <recieveMoney/>
              </v-tab-item>
              <v-tab-item>
                <sendMoney/>
              </v-tab-item>
              <v-tab-item>
                <transactions/>
              </v-tab-item>
           </v-tabs>
          <v-divider/>
          <v-card-actions>
          <v-btn
            color="orange"
            text
          >
            <v-icon>mdi-help</v-icon>
          </v-btn>
          <v-btn
            color="primary"
            text
            @click='refresh()'
          >
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import {
  listWalletsThatExist, checkIfNodeProcessRunning, startDeamon,
  listLoadedWallets, loadWallet, permissionElectrum
} from '@/assets/util/btc/electrum/general.js'
import { mapState, mapGetters } from 'vuex'
import balance from '@/components/btcWallet/singleSig/balance/balance.vue'
import sendMoney from '@/components/btcWallet/singleSig/sendMoney/main.vue'
import recieveMoney from '@/components/btcWallet/singleSig/recMoney/mainCard.vue'
import transactions from '@/components/btcWallet/singleSig/transactions/mainCard.vue'
export default {
  components: {
    balance,
    sendMoney,
    recieveMoney,
    transactions
  },
  data: () => ({
    walletReady: false,
    retries: 10,
    walletError: false,
    correctWalletExist: true,
    keyInfo: 1
  }),
  computed: {
    ...mapState('bitcoinInfo', [
      'btcSingleSigTestnet'
    ]),
    network: function () {
      return this.btcSingleSigTestnet.network
    },
    ...mapGetters('hardwareInfo', [
      'singleSigElectrumName',
      'singleSigHardwareWalletInfo'
    ])
  },
  methods: {
    refresh: function () {
      this.keyInfo = this.keyInfo * -1
    },
    start: async function () {
      if (this.retries < 0) {
        throw Error('Too many attempts to bring up wallet')
      }
      if (!this.correctWalletExist) {
        return false
      }
      let nextready = await this.checkCorrectWalletExist()
      if (nextready) {
        nextready = await this.electrumRunning()
      }
      if (nextready) {
        nextready = await this.correctWalletLoaded()
      }
      if (!nextready) {
        return false
      }
      return true
    },
    electrumRunning: async function () {
      const nodeStatus = await checkIfNodeProcessRunning()
      if (!nodeStatus) {
        this.retries = this.retries - 1
        await startDeamon(this.network)
        this.start()
      }
      return true
    },
    checkCorrectWalletExist: async function () {
      const wallets = await listWalletsThatExist(this.network)
      if (!wallets.includes(this.singleSigElectrumName)) {
        this.correctWalletExist = false
        return false
      }
      return true
    },
    correctWalletLoaded: async function () {
      const loadedWallets = await listLoadedWallets(this.btcSingleSigTestnet.rpcPort,
        this.btcSingleSigTestnet.rpcUser, this.btcSingleSigTestnet.rpcPassword)
      if (!loadedWallets.data.result.includes(this.singleSigElectrumName)) {
        this.retries = this.retries - 1
        await loadWallet(this.singleSigElectrumName, this.btcSingleSigTestnet.rpcPort,
          this.btcSingleSigTestnet.rpcUser, this.btcSingleSigTestnet.rpcPassword,
          this.network)
        this.start()
      }
      return true
    }
  },
  async mounted () {
    try {
      await permissionElectrum()
      const startSuccessful = await this.start()
      if (startSuccessful) {
        this.walletReady = true
      } else {
        console.log('wallet not ready')
      }
    } catch (e) {
      this.walletError = true
      console.log('Wallet Bring Up Error')
    }
  }
}
</script>
<style scoped>

</style>
