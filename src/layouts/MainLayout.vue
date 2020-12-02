<template>
  <q-layout view="lhh lpR lff">
    <q-header :class="($q.dark.isActive?'text-white':'text-black') + ' bg-transparent q-pa-sm'">
      <q-toolbar>

        <q-btn dense flat round icon="menu" @click="left = !left" v-if="!left" />

        <q-toolbar-title class="logo flex">
            <img v-if="(!left)&&(!$q.dark.isActive)" src="~/assets/logo-blue.svg" height="32">
            <img v-if="(!left)&&$q.dark.isActive" src="~/assets/logo-white.svg" height="32">
            <span class="q-ml-sm">
                aleph.im
            </span>
        </q-toolbar-title>
        <q-space />
        <q-btn-group class="shadow-1 bg-aleph-radial">
          <q-btn v-if="!account" size="md" class="bg-aleph-radial text-white" @click="web3Connect">Connect to a wallet</q-btn>
          <q-btn v-if="account" class="text-white">
            {{balance_info['ALEPH'].toFixed(2)}} <img src="~/assets/logo-white.svg" style="height: 1.4em; margin: 0 0 .2em .4em;"/>
          </q-btn>
          <q-btn v-if="account" color="white" text-color="black" class="rounded-forced">{{ellipseAddress(account.address)}}</q-btn>
        </q-btn-group>
      </q-toolbar>
    </q-header>

    <q-drawer show-if-above v-model="left" side="left" content-class="column justify-between" :width="200">
      <!-- drawer content -->
      <div>
        <p class="q-pa-md">
          <img v-if="!$q.dark.isActive" src="~/assets/logo-blue.svg" height="32">
          <img v-else src="~/assets/logo-white.svg" height="32">
        </p>
        <q-list padding class="menu">
          <q-item v-for="link of links1"
                  :key="link.text"
                  clickable v-ripple
                  :to="link.link" :exact="link.exact">
            <q-item-section avatar v-if="link.icon">
              <q-icon :name="link.icon" />
            </q-item-section>

            <q-item-section>
              {{link.text}}
            </q-item-section>
          </q-item>
        </q-list>
      </div>
      <div>
        <q-list>
          <q-item tag="label" v-ripple class="opacity">
            <q-item-section>
              <q-item-label>Dark mode</q-item-label>
            </q-item-section>
            <q-item-section avatar>
              <q-toggle :value="$q.dark.isActive" @input="$q.dark.set(!$q.dark.isActive)" />
            </q-item-section>
          </q-item>
        </q-list>
      </div>
    </q-drawer>
    <q-page-container>
      <router-view />
    </q-page-container>

    <q-footer :class="($q.dark.isActive?'text-white':'text-black') + ' bg-transparent q-pa-sm q-pt-lg'">
      Copyright ©2020-present <a href="https://aleph.im/">aleph.im project</a>, all rights reserved.
    </q-footer>

  </q-layout>
</template>
<script>
import Web3Modal from 'web3modal'
import WalletConnectProvider from '@walletconnect/web3-provider'
import Portis from '@portis/web3'
import MewConnect from '@myetherwallet/mewconnect-web-client'

import { ellipseAddress } from '../helpers/utilities'

import { mapState } from 'vuex'
import {
  ethereum
} from 'aleph-js'

export default {
  name: 'MainLayout',

  components: {
  },
  computed: mapState({
    account: state => state.account,
    balance_info: state => state.balance_info,
    api_server: state => state.api_server
  }),
  watch: {
    async account () {
      await this.$store.dispatch('update_balance')
    }
  },
  data () {
    return {
      ellipseAddress: ellipseAddress,
      left: false,
      search: '',
      storage: 0,
      show_backup: false,
      lbimgs: '', // Img Url , string or Array
      lbvisible: false,
      lbidx: 0, // default: 0,
      display_onboarding: false,
      links1: [
        { text: 'Dashboard', link: { name: 'dashboard' }, exact: true },
        { text: 'Nodes and Staking', link: { name: 'stake' }, exact: false },
        { text: 'Swap', link: { name: 'swap' }, exact: false }
        // { icon: 'far fa-newspaper', text:'My Feed' },
        // { icon: 'photo', text: 'Photos' },
        // { icon: 'people', text: 'Contacts' }
      ],
      links2: [
        // { icon: 'archive', text: 'Archive' },
        // { icon: 'delete', text: 'Trash' }
      ],
      links3: [
        // { icon: 'settings', text: 'Settings' },
        // { icon: 'help', text: 'Help & Feedback' },
        // { icon: 'get_app', text: 'App Downloads' }
      ],
      createMenu: [
        // { icon: 'photo_album', text: 'Album' },
        // { icon: 'people', text: 'Shared Album' },
        // { icon: 'movie', text: 'Movie' },
        // { icon: 'library_books', text: 'Animation' },
        // { icon: 'dashboard', text: 'Collage' },
        // { icon: 'book', text: 'Photo book' }
      ]
    }
  },
  methods: {
    async web3Connect () {
      const providerOptions = {
        /* See Provider Options Section */
        walletconnect: {
          package: WalletConnectProvider, // required
          options: {
            infuraId: '4890a5bd89854916b128088119d76b50' // required
          }
        },
        portis: {
          package: Portis, // required
          options: {
            id: '9e6ad33b-ef5c-49a1-b39c-e664dfc862d1' // required
          }
        },
        mewconnect: {
          package: MewConnect, // required
          options: {
            infuraId: '4890a5bd89854916b128088119d76b50' // required
          }
        }
      }

      const web3Modal = new Web3Modal({
        network: 'mainnet', // optional
        cacheProvider: false, // optional
        providerOptions // required
      })

      const provider = await web3Modal.connect()

      let account = await ethereum.from_provider(provider)
      // let accounts = await this.w3.eth.getAccounts()
      if (account) {
        this.$store.commit('set_account', account)
      }
    }
  }
}
</script>

<style lang="scss">
.logo {
  .q-badge {
    position: absolute;
    margin-top: -0.2rem;
    margin-left: -1rem;
  }
}

.q-drawer {
  background: $light-grey;
}

.q-drawer--dark {
  background: #172025;

  .q-list.menu {
    .q-item {
      opacity: 0.5;
      color: #F6F8FB;

      &.q-router-link--active {
        color: #fff;
        opacity: 1.0;
      }
    }
  }
}

.content {
  max-width: 980px;
  margin: 0 auto;
  padding: 0 10px;
  word-wrap: normal;

  .title {
    font-size: 3rem;

    @media (max-width: $breakpoint-sm-max) {
      font-size: 4.4vw;
    }
  }

  .menu {
    background: #7a40f2;
    border-radius: 20px !important;
    margin: 10px;
    padding: 10px;
  }
}

#left {
  .q-drawer {
    :first-child {
      border-radius: 15px;
    }
    padding: 5px !important;

    .q-item.q-item--active {
      background: #fff;
      margin-left: 10px;
      padding-right: 10px;
      border-radius: 15px 0 0 15px;

    border-top-right-radius: 0;
    border-bottom-right-radius: 0;

      &:before, &:after{
        box-sizing: content-box;
        content: '';
        position: absolute;
        left: 100%; /* I use this instead of right: 0 to avoid 1px rounding errors */
        margin-left: -15px; /* I use this because I am using left: 100% */
        width: 15px;
        height: 15px;
        border-right: 15px solid #fff ;
        z-index: 10;
      }

      &:before {
        top: -15px;
        border-bottom: 15px solid  #fff;
        border-bottom-right-radius: 30px;
      }

      &:after {
        bottom: -15px;
        border-top: 15px solid  #fff;
        border-top-right-radius: 30px;
      }
    }
  }
}

.rounded-large {
  border-radius: 25px;
}

.rounded-forced {
  border-radius: 10px !important;
}
</style>