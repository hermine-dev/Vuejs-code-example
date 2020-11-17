<template>
  <div>
    <app-breadcrumb title="Dashboard"></app-breadcrumb>
    <v-container fluid grid-list-xl pa-0>
      <v-layout row wrap>
        <!--Competitors Coverage Start-->
        <v-flex lg4 md6 sm6 xs12>
          <app-card maxHeight="540px">
            <competitors-coverage chartHeight="380"/>
          </app-card>
        </v-flex>
        <!--Competitors Coverage End-->

        <!--Price Alterations Start-->
        <v-flex lg4 md6 sm6 xs12>
          <app-card maxHeight="540px">
            <price-alternations chartHeight="345" />
          </app-card>
        </v-flex>
        <!--Price Alterations End-->

        <!--Fast Moving Products Start-->
        <v-flex lg4 md6 sm6 xs12>
          <app-card maxHeight="540px">
            <fast-moving />
          </app-card>
        </v-flex>
        <!--Fast Moving Products End-->
      </v-layout>

      <!--Data Table Start-->
      <v-layout row wrap>
        <v-flex xs12>
          <app-card>
            <live-price-log />
          </app-card>
        </v-flex>
      </v-layout>
      <!--Data Table End-->
    </v-container>
  </div>
</template>

<script>
import CompetitorsCoverage from '../components/dashboard/competitors-coverage/Container'
import PriceAlternations from '../components/dashboard/price-alternations/Container'
import FastMoving from '../components/dashboard/fast-moving/Container'
import LivePriceLog from '../components/dashboard/live-price-log/Container'
export default {
  name: 'Dashboard',
  components: {
    CompetitorsCoverage,
    PriceAlternations,
    FastMoving,
    LivePriceLog
  },
  data: () => ({
    window: {
      width: 0,
      height: 0
    }
  }),
  computed: {
    getColorScheme () {
      return this.$store.state.layout.colorScheme
    }
  }
}
</script>

<style scoped>

</style>
