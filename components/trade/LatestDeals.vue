<template lang="pug">
// Переделать как табличку element
.deals-history(v-loading="loading")
  .blist
    .ltd.d-flex.justify-content-around
      span
      span.text-muted Latest Deals
      span

    .ltd.d-flex.justify-content-around.mt-1
      span Price ({{ quote_token.symbol.name }})
      span Amount ({{ base_token.symbol.name }})
      span Time

  .orders-list.blist
    a(v-for="deal in coloredDeals" :href="monitorTx(deal.trx_id)" target="_blank")
      .ltd.d-flex.justify-content-around
        span(:class="deal.cls")  {{ deal.unit_price | humanPrice(6) }}
        span {{ deal.amount | humanFloat(base_token.symbol.precision) }}
        span {{ deal.time | moment('DD-MM HH:mm')}}

</template>

<script>
import { mapState } from 'vuex'

export default {
  async fetch() {
    await this.$store.dispatch('market/fetchDeals')
  },

  data() {
    return {
      loading: false
    }
  },

  computed: {
    ...mapState('market', ['deals', 'quote_token', 'base_token', 'id']),
    ...mapState(['network']),

    coloredDeals() {
      return Array.from(this.deals)
        .sort((a, b) => b.time - a.time).map(h => {
          if (h.type == 'buymatch') {
            h.cls = 'text-success'
            h.amount = h.bid.amount
          } else {
            h.cls = 'text-danger'
            h.amount = h.ask.amount
          }

          return h
        })
    }
  },

  watch: {
    id() {
      this.fetch()
    }
  },

  mounted() {
    this.fetch()
  },

  methods: {
    async fetch() {
      this.loading = true

      try {
        await this.$store.dispatch('market/fetchDeals')
      } catch (e) {
        this.$notify({ title: 'Fetch deals', message: e, type: 'error' })
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style lang="scss">
.blist a {
  all: unset;
}

.deals-history {
  .orders-list {
    height: 250px;
  }
}
</style>
