<template lang="pug">
  .nft-wallet
    small.text-muted(v-if="nfts.length == 0") There is no NFT's yet..
    el-card(v-for="nft in nfts").mb-2
      .float-right
        el-button(size="mini" type="primary" @click="send(nft)").mb-1 Send
          i.el-icon-s-promotion.ml-2

      .row
        .col-lg-3
          img(:src="nft.mdata.img" height="150").order-img
        .col-lg-9
          NftFields(:nft="nft")
</template>

<script>
import { mapGetters, mapState } from 'vuex'

import { prepareNFT } from '~/utils'

import NftFields from '~/components/nft_markets/NftFields'

export default {
  components: {
    NftFields
  },

  data() {
    return {
      search: '',

      nfts: []
    }
  },

  computed: {
    ...mapGetters(['user']),
    ...mapGetters('api', ['rpc']),
  },

  watch: {
    user() {
      this.fetchNfts()
    }
  },

  mounted() {
    this.fetchNfts()
  },

  methods: {
    send(nft) {
      this.$prompt('Please input account where you want transfer NFT', 'lol', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel'
      }).then(async ({ value }) => {
        const authorization = [this.user.authorization]
        const actions = []

        actions.push({
          account: 'simpleassets',
          name: 'transfer',
          authorization,
          data: {
            from: this.user.name,
            to: value,
            assetids: [nft.id],
            memo: 'alcor.exchange/wallet/nfts'
          }
        })

        const loading = this.$loading({
          lock: true,
          text: 'Wait for wallet'
        })

        try {
          await this.$store.dispatch('chain/sendTransaction', actions)

          this.$store.dispatch('market/fetchOrders')
          this.$notify({ title: 'Send NFT', message: 'NFT sent successfully!', type: 'success' })
          this.fetchNfts()
        } catch (e) {
          this.$notify({ title: 'Send NFT', message: e, type: 'error' })
          console.log(e)
        } finally {
          loading.close()
        }
      })
    },

    async fetchNfts() {
      console.log('fetchNfts', this.user.name)
      if (!this.user) return

      const { rows } = await this.rpc.get_table_rows({
        code: 'simpleassets',
        scope: this.user.name,
        table: 'sassets',
        limit: 1000
      })

      rows.map(n => {
        prepareNFT(n)
      })

      console.log(rows)
      this.nfts = rows
    }
  }
}

</script>

<style lang="scss">
</style>
