<template>
  <div>
    <analytics-filter
      ref="filter"
      :open="filter.open"
      :data="filterData"
      :savedFilters="getSavedFilters"
      :sellers="getFilterSellers"
      :brands="getFilterBrands"
      :categories="getFilterCategories"
      @deleteSavedFilter="deleteSavedFilter($event)"
      @savedFilterSelected="apply()">
      <template v-slot:toggle>
        <button class="toggle-btn" @click="toggleFilter()"><v-icon>icon-levels-1</v-icon> Filter</button>
      </template>
      <template v-slot:header>
        <span>Analytics Filter</span>
        <button @click="toggleFilter()"><v-icon>close</v-icon></button>
      </template>
      <template v-slot:footer>
        <v-btn class="text-normal i-v-btn shadow-0 ml-0" :ripple="false" light>Load</v-btn>
        <v-btn @click="apply()" class="text-normal i-v-btn shadow-0 ml-0" :ripple="false" color="success">Apply</v-btn>
        <v-btn @click="saveHandler()" class="text-normal i-v-btn shadow-0 ml-0" :ripple="false" color="primary">Save</v-btn>
      </template>
    </analytics-filter>
    <sticky-breadcrumb>
      <v-layout row wrap>
        <v-flex sm6 xs12 py-0>
          <h2>Analytics</h2>
          <p>Filters: </p>
          <template v-if="getFilterBrandsREQ.length || getFilterSellersREQ.length || getFilterCategoriesREQ.length">
            <p v-if="getFilterSellersREQ.length">Sellers ({{getFilterSellersREQ.length}}) <button><v-icon>close</v-icon></button></p>
            <p v-if="getFilterCategoriesREQ.length">Categories ({{getFilterCategoriesREQ.length}}) <button><v-icon>close</v-icon></button></p>
            <p v-if="getFilterBrandsREQ.length">Brands ({{getFilterBrandsREQ.length}}) <button><v-icon>close</v-icon></button></p>
          </template>
          <p v-else>All</p>
        </v-flex>
        <v-flex sm6 xs12 py-0>
        </v-flex>
      </v-layout>
    </sticky-breadcrumb>
    <v-container fluid grid-list-xl pa-0>
      <!--Average deviation Start-->
      <v-layout row wrap>
        <v-flex lg8 md8>
          <app-card>
            <average-deviation-chart
              :chartOptions="getAverageDeviation"
              @downloadExcel="downloadExcelHandler({
                task: 'pricesDeviation',
                downloadExcel: true,
                payload: {
                  sellers: getFilterSellersREQ,
                  categories: getFilterCategoriesREQ,
                  brands: getFilterBrandsREQ
                }
              })" />
          </app-card>
        </v-flex>
        <v-flex lg4 md4>
          <app-card>
            <sellers-ranking
              :filters="filterData"
              :data="getSellersRanking" />
          </app-card>
        </v-flex>
      </v-layout>
      <!--Average deviation End-->

      <!--Profit Opportunities Start-->
      <v-layout row wrap>
        <v-flex lg9 md8>
          <app-card>
            <profit-opportunities
              :table="getProfitOpportunities.table"
              :filters="getProfitOpportunities.filters"
              :categories="profitOpportunities.categories"
              :brands="profitOpportunities.brands"
              @downloadExcel="downloadExcelHandler({
                task: 'profitOpportunities',
                downloadExcel: true,
                dropdown: getNestedItems(getProfitOpportunities.filters),
                payload: {
                  sellers: getFilterSellersREQ,
                  categories: getFilterCategoriesREQ,
                  brands: getFilterBrandsREQ
                }
              })" />
          </app-card>
        </v-flex>
        <v-flex lg3 md4>
          <app-card>
            <price-distribution
              :filters="filterData"
              @downloadExcel="downloadExcelHandler({
                task: 'pricesDistribution',
                downloadExcel: true,
                payload: {
                  name: $event.name,
                  filters: {
                    sellers: getFilterSellersREQ,
                    categories: getFilterCategoriesREQ,
                    brands: getFilterBrandsREQ
                  }
                }
              })" />
          </app-card>
        </v-flex>
      </v-layout>
      <!--Profit Opportunities End-->

      <!--Competitors Coverage Start-->
      <v-layout row wrap>
        <v-flex xs12>
          <app-card>
            <competitors-coverage
              chartHeight="380" />
          </app-card>
        </v-flex>
      </v-layout>
      <!--Competitors Coverage End-->

      <v-layout row wrap>
        <v-flex lg6>
          <app-card>
            <sellers-category-distribution
              :table="getSellersCategoryDistribution.table"
              :filters="getSellersCategoryDistribution.filters"
              :categories="sellersDataDistributionCategories"
              :sellers="sellersDataDistributionSellers"
              @downloadExcel="downloadExcelHandler({
                task: 'sellersCategoryDataDistribution',
                downloadExcel: true,
                dropdown: getNestedItems(getSellersCategoryDistribution.filters),
                payload: {
                  sellers: getFilterSellersREQ,
                  categories: getFilterCategoriesREQ,
                  brands: getFilterBrandsREQ
                }
              })" />
          </app-card>
        </v-flex>
        <v-flex lg6>
          <app-card>
            <sellers-brand-distribution
              :table="getSellersBrandDistribution.table"
              :filters="getSellersBrandDistribution.filters"
              :brands="sellersDataDistributionBrands"
              :sellers="sellersDataDistributionSellers2"
              @downloadExcel="downloadExcelHandler({
                task: 'sellersBrandDataDistribution',
                downloadExcel: true,
                dropdown: getNestedItems(getSellersBrandDistribution.filters),
                payload: {
                  sellers: getFilterSellersREQ,
                  categories: getFilterCategoriesREQ,
                  brands: getFilterBrandsREQ
                }
              })" />
          </app-card>
        </v-flex>
      </v-layout>
    </v-container>

    <!-- Save Dialog-->
    <v-dialog
      v-model="saveModal.show"
      transition="slide-x-transition"
      content-class="i-dialog"
      width="500"
      :dark="getThemeColor === '0'"
      attach
    >
      <v-card>
        <v-card-title
          class="headline"
          primary-title
        >
          Save {{ !getSavedFilters.model ? 'New ' : null }} Filter
          <v-btn
            class="modal-close-btn"
            :ripple="false"
            small
            flat
            solo
            @click="saveModal.show = false"
          >
            <v-icon>close</v-icon>
          </v-btn>
        </v-card-title>
        <v-form
          ref="saveFilterForm"
          lazy-validation
          @submit.prevent="saveFilterRequest()"
          class="py-3"
        >
          <v-card-text class="dialog-dropdowns-list py-0" v-if="getSavedFilters.model"
          >
            <v-layout>
              <v-flex xs6>
                <p>Old Name</p>
              </v-flex>
              <v-flex xs6 py-0>
                <v-text-field
                  v-model="getSavedFilters.model"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  disabled
                  placeholder="Old Name"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                ></v-text-field>
              </v-flex>
            </v-layout>
          </v-card-text>
          <v-card-text class="dialog-dropdowns-list py-0">
            <v-layout>
              <v-flex xs6>
                <p>New Name</p>
              </v-flex>
              <v-flex xs6 py-0>
                <v-text-field
                  v-model="saveModal.newName"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Enter Filter Name"
                  validate-on-blur
                ></v-text-field>
              </v-flex>
            </v-layout>
            <div class="text-xs-right">
              <v-btn
                type="submit"
                class="i-btn-hoverable text-normal mx-0 mt-3"
                flat
                color="primary"
                :ripple="false"
              >
                Save
              </v-btn>
            </div>
          </v-card-text>
        </v-form>
      </v-card>
    </v-dialog>

    <!--Delete Dialog-->
    <v-dialog
      v-model="deleteModal.show"
      transition="slide-x-transition"
      content-class="i-dialog"
      width="500"
      :dark="getThemeColor === '0'"
      attach
    >
      <v-card>
        <v-btn icon dark flat :ripple="false" class="close-icon-absolute" @click="cancelDeleteSavedFilter()">
          <v-icon>close</v-icon>
        </v-btn>
        <v-card-text class="delete-popup">
          <img src="../assets/img/icons/delete.svg" alt="">
          <h2>Are you sure ?</h2>
          <p>You are about to delete the filter.</p>
          <div class="btn-group justify-center">
            <v-btn
              class="text-normal i-v-btn shadow-0 ml-0"
              :ripple="false"
              color="transparent"
              @click="cancelDeleteSavedFilter()">
              Cancel
            </v-btn>
            <v-btn
              class="text-normal i-v-btn shadow-0 ml-0"
              :ripple="false"
              color="primary"
              @click="deleteSavedFilterRequest()">
              Accept
            </v-btn>
          </div>
        </v-card-text>
      </v-card>
    </v-dialog>

    <!-- Save Toaster-->
    <notifications group="save" position="bottom right" :max="toaster.max" width="350">
      <template slot="body" slot-scope="props">
        <div class="default-notification save-notification">
          <a class="close" @click="props.close">
            <v-icon>close</v-icon>
          </a>
          <a class="title">
            <img src="../assets/img/icons/success.svg" alt="">
            {{ props.item.title }}
          </a>
          <p v-html="props.item.text"></p>
        </div>
      </template>
    </notifications>

    <!--Delete Toaster-->
    <notifications group="delete" position="bottom right" :max="toaster.max" width="350">
      <template slot="body" slot-scope="props">
        <div class="default-notification save-notification">
          <a class="close" @click="props.close">
            <v-icon>close</v-icon>
          </a>
          <a class="title danger">
            <img src="../assets/img/icons/cancel.svg" alt="">
            {{ props.item.title }}
          </a>
          <p v-html="props.item.text"></p>
        </div>
      </template>
    </notifications>
  </div>
</template>

<script>
import { AppUrl } from '@/assets/js/variables'
import AverageDeviationChart from '../components/analytics/average-deviation/Chart'
import CompetitorsCoverage from '../components/analytics/competitors-coverage/parentComponent'
import profitOpportunities from '../components/analytics/profitOpportunities'
import priceDistribution from '../components/analytics/priceDistribution'
import sellersCategoryDistribution from '../components/analytics/sellersCategoryDistribution'
import sellersBrandDistribution from '../components/analytics/sellersBrandDistribution'
import sellersRanking from '../components/analytics/sellersRanking'
import analyticsFilter from '../components/analytics/filter'
export default {
  name: 'Analytics',
  components: {
    AverageDeviationChart,
    CompetitorsCoverage,
    profitOpportunities,
    priceDistribution,
    sellersCategoryDistribution,
    sellersBrandDistribution,
    sellersRanking,
    analyticsFilter
  },
  data: () => ({
    profitOpportunities: {
      categories: {
        search: null,
        isLoading: false
      },
      brands: {
        search: null,
        isLoading: false
      }
    },
    competitorsCoverage: {
      height: 400
    },
    sellersDataDistributionCategories: {
      isLoading: false,
      items: [],
      model: null,
      search: null
    },
    sellersDataDistributionSellers: {
      isLoading: false,
      items: [],
      model: null,
      search: null
    },
    sellersDataDistributionBrands: {
      isLoading: false,
      items: [],
      model: null,
      search: null
    },
    sellersDataDistributionSellers2: {
      isLoading: false,
      items: [],
      model: null,
      search: null
    },
    filterData: {
      category: {
        isLoading: false,
        items: [],
        model: [],
        search: null,
        optionsIsLoading: false
      },
      brand: {
        isLoading: false,
        items: [],
        model: [],
        search: null,
        optionsIsLoading: false
      },
      seller: {
        isLoading: false,
        items: [],
        model: [],
        search: null,
        optionsIsLoading: false
      }
    },
    filter: {
      media: false,
      open: false
    },
    saveModal: {
      show: false,
      newName: ''
    },
    deleteModal: {
      show: false,
      deleteItem: null
    },
    toaster: {
      max: 3
    }
  }),
  mounted () {
    if (!this.getFilterSellersREQ.length || !this.getFilterCategoriesREQ.length || !this.getFilterBrandsREQ.length) {
      this.requestFilterSellers()
      this.requestFilterBrands()
      this.requestFilterCategories()
      this.getSavedFiltersRequest()
    }
  },
  methods: {
    requestFilterSellers () {
      if (this.getFilterSellers.items) return
      this.filterData.seller.optionsIsLoading = true
      this.$store.dispatch('analyticsFilterSellerModule/getFilterSellers', {
        pageId: 'analytics'
      }).then(() => {
        this.filterData.seller.optionsIsLoading = false
      })
    },
    requestFilterBrands () {
      if (this.getFilterBrands.items) return
      this.filterData.brand.optionsIsLoading = true
      this.$store.dispatch('analyticsFilterSellerModule/getFilterBrands', {
        pageId: 'analytics'
      }).then(() => {
        this.filterData.brand.optionsIsLoading = false
      })
    },
    requestFilterCategories () {
      if (this.getFilterCategories.items) return
      this.filterData.category.optionsIsLoading = true
      this.$store.dispatch('analyticsFilterSellerModule/getFilterCategories', {
        pageId: 'analytics'
      }).then(() => {
        this.filterData.category.optionsIsLoading = false
      })
    },
    getSavedFiltersRequest () {
      this.$store.dispatch('analyticsFilterSellerModule/getSavedFilters')
    },
    resetFilter () {
      this.$store.commit('analyticsFilterSellerModule/RESET_filter')
    },
    saveHandler () {
      if (this.getFilterSellersREQ.length || this.getFilterCategoriesREQ.length || this.getFilterBrandsREQ.length) {
        this.saveModal.show = true
      }
    },
    saveFilterRequest () {
      this.$store.dispatch('analyticsFilterSellerModule/saveFilter', {
        old_name: this.getSavedFilters.model ? this.getSavedFilters.model : this.saveModal.newName,
        new_name: this.saveModal.newName ? this.saveModal.newName : this.getSavedFilters.model,
        sellers: this.getFilterSellersREQ,
        categories: this.getFilterCategoriesREQ,
        brands: this.getFilterBrandsREQ
      }).then(() => {
        this.$notify({
          group: 'save',
          title: 'New Filter',
          text: 'New Filter Saved',
          duration: 500,
          speed: 600
        })
        this.getSavedFiltersRequest()
        this.saveModal.show = false
      })
    },
    apply () {
      this.updateAverageDeviation()
      this.updateSellersRanking()
      this.updateProfitOpportunities()
      this.updatePriceDistribution()
      this.updateSellersDataDistribution()
      this.updateSellersBrandDistribution()
      this.updateCompetitorsCoverage()
      this.toggleFilter()
    },
    deleteSavedFilter (name) {
      this.deleteModal.show = true
      this.deleteModal.deleteItem = name
    },
    cancelDeleteSavedFilter () {
      this.deleteModal.show = false
      this.deleteModal.deleteItem = null
    },
    deleteSavedFilterRequest () {
      this.$refs.filter.getBySavedFilter([], false)
      this.$store.dispatch('analyticsFilterSellerModule/deleteSavedFilter', this.deleteModal.deleteItem).then(() => {
        this.cancelDeleteSavedFilter()
        this.$notify({
          group: 'delete',
          title: 'Deleted',
          text: 'The filter has been deleted',
          duration: 500,
          speed: 600
        })
        this.getSavedFiltersRequest()
        this.$store.dispatch('analyticsFilterSellerModule/getBySavedFilter', [])
      })
    },
    toggleFilter () {
      this.filter.open = !this.filter.open
    },
    updateAverageDeviation () {
      this.$store.dispatch('analyticsAverageDeviationModule/getChart', {
        sellers: this.getFilterSellersREQ,
        categories: this.getFilterCategoriesREQ,
        brands: this.getFilterBrandsREQ
      })
    },
    updateSellersRanking () {
      this.$store.dispatch('analyticsSellersRankingModule/getTable', {
        sellers: this.getFilterSellersREQ,
        categories: this.getFilterCategoriesREQ,
        brands: this.getFilterBrandsREQ
      })
    },
    updateCompetitorsCoverage () {
      this.$store.dispatch('analyticsCompetitorsCoverageModule/getChart', {
        brandId: this.getFilterBrandsREQ,
        sellerId: this.getFilterSellersREQ,
        categoryId: this.getFilterCategoriesREQ
      })
    },
    updatePriceDistribution () {
      this.$store.dispatch('analyticsPriceDistributionModule/getChart', {
        sellers: this.getFilterBrandsREQ,
        categories: this.getFilterSellersREQ,
        brands: this.getFilterCategoriesREQ
      })
    },
    updateProfitOpportunities () {
      this.$store.dispatch('analyticsProfitOpportunitiesModule/getProfitOpportunities', {
        sellers: this.getFilterSellersREQ,
        categories: this.getFilterCategoriesREQ,
        brands: this.getFilterBrandsREQ
      })
    },
    updateSellersDataDistribution () {
      this.$store.dispatch('analyticsSellersCategoryDistributionModule/getSellersDataDistribution', {
        sellers: this.getFilterSellersREQ,
        categories: this.getFilterCategoriesREQ,
        brands: this.getFilterBrandsREQ
      })
    },
    updateSellersBrandDistribution () {
      this.$store.dispatch('analyticsSellersBrandDistributionModule/getSellersDataDistribution', {
        sellers: this.getFilterSellersREQ,
        categories: this.getFilterCategoriesREQ,
        brands: this.getFilterBrandsREQ
      })
    },
    downloadExcelHandler (params) {
      console.log('Params', params)
      let form = document.createElement('form')
      form.setAttribute('method', 'post')
      form.setAttribute('action', AppUrl)
      form.setAttribute('target', 'targetForm')
      for (let i in params) {
        if (params.hasOwnProperty(i)) {
          let input = document.createElement('input')
          input.type = 'hidden'
          input.name = i
          input.value = JSON.stringify(params[i])
          form.appendChild(input)
        }
      }
      document.body.appendChild(form)
      window.open('', 'targetForm')
      form.submit()
      form.remove()
    },
    getNestedItems (obj) {
      let newObj = {}
      for (let key in obj) {
        Object.assign(newObj, {[key]: obj[key].checked})
      }
      return newObj
    }
  },
  computed: {
    getAverageDeviation () {
      return this.$store.state.analyticsAverageDeviationModule.chart
    },
    getFilterSellers () {
      return this.$store.state.analyticsFilterSellerModule.sellers
    },
    getSellersRanking () {
      return this.$store.state.analyticsSellersRankingModule
    },
    getFilterCategoriesREQ () {
      return this.$store.state.analyticsFilterSellerModule.categories.request
    },
    getFilterSellersREQ () {
      return this.$store.state.analyticsFilterSellerModule.sellers.request
    },
    getFilterBrandsREQ () {
      return this.$store.state.analyticsFilterSellerModule.brands.request
    },
    getFilterBrands () {
      return this.$store.state.analyticsFilterSellerModule.brands
    },
    getFilterCategories () {
      return this.$store.state.analyticsFilterSellerModule.categories
    },
    getProfitOpportunities () {
      return this.$store.state.analyticsProfitOpportunitiesModule
    },
    getSellersCategoryDistribution () {
      return this.$store.state.analyticsSellersCategoryDistributionModule
    },
    getSellersBrandDistribution () {
      return this.$store.state.analyticsSellersBrandDistributionModule
    },
    getSavedFilters () {
      return this.$store.state.analyticsFilterSellerModule.savedFilters
    },
    getThemeColor () {
      return this.$store.state.auth.userInfo.themeColor
    }
  },
  destroyed () {
    this.resetFilter()
  }
}
</script>

<style scoped>

</style>
