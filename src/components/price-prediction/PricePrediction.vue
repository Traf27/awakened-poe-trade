<template>
  <div :style="{ 'min-height': !showContrib ? '70px' : undefined }">
    <div v-if="loading" class="text-center py-2">
      <i class="fas fa-dna fa-spin text-gray-600"></i>
      <span class="pl-2">Getting price prediction...</span>
    </div>
    <div v-else-if="price">
      <div class="flex items-center pb-4">
        <div class="flex items-center justify-center flex-1">
          <div class="w-8 h-8 flex items-center justify-center">
            <img :src="item.computed.icon" :alt="item.name" class="max-w-full max-h-full">
          </div>
          <i class="fas fa-arrow-right text-gray-600 px-2"></i>
          <span class="px-1 text-base">
            <span v-if="price.min === price.max" class="text-gray-600 font-sans">~&nbsp;</span><span>{{ price.min }}</span>
            <template v-if="price.min !== price.max"><span class="text-gray-600 font-sans">&nbsp;~&nbsp;</span>{{ price.max }}</template> ×</span>
          <div class="w-8 h-8 flex items-center justify-center">
            <img :src="currency.icon" :alt="currency.name" class="max-w-full max-h-full">
          </div>
        </div>
        <div class="text-center">
          <div class="leading-tight">
            <i v-if="price.confidence < 70" class="fas fa-exclamation-circle pr-1 text-orange-500"></i>
            <span>{{ price.confidence }}&nbsp;%</span>
          </div>
          <div class="text-xs text-gray-500 leading-none">Confidence</div>
        </div>
      </div>
      <div v-if="!showContrib" class="flex justify-between items-center">
        <button @click="showContrib = true" class="btn">Contribution to predicted price<i class="fas fa-chevron-down btn-icon ml-2"></i></button>
        <a href="https://www.poeprices.info/" @click.prevent="openWebsite" class="text-xs text-gray-700">poeprices.info</a>
      </div>
      <table v-else>
        <thead>
          <th></th>
          <th class="text-gray-500 text-left font-normal pl-2">
            <button @click="showContrib = false">Contribution to predicted price<i class="fas fa-chevron-up btn-icon ml-2"></i></button>
          </th>
        </thead>
        <tbody class="align-top">
          <tr v-for="expl in price.explanation" :key="expl.name">
            <td class="text-right text-gray-500">{{ expl.contrib }}&nbsp;%</td>
            <td class="pl-2 truncate w-full" style="max-width: 0;">{{ expl.name }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-else-if="error">
      <i class="fas fa-exclamation-circle pr-1 text-red-600"></i>
      <span>{{ error }}</span>
    </div>
  </div>
</template>

<script>
import { shell } from 'electron'
import { requestPoeprices } from './poeprices'
import { Prices } from '../Prices'

export default {
  name: 'PricePrediction',
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      price: null,
      error: null,
      loading: false,
      showContrib: false
    }
  },
  watch: {
    item: {
      immediate: true,
      async handler (item) {
        try {
          this.loading = true
          this.error = null
          this.price = null
          this.showContrib = false
          this.price = await requestPoeprices(this.item)
        } catch (err) {
          this.error = err.message
        } finally {
          this.loading = false
        }
      }
    }
  },
  computed: {
    currency () {
      return Prices.findByDetailsId(this.price.currency)
    }
  },
  methods: {
    openWebsite (e) {
      shell.openExternal(e.target.href)
    }
  }
}
</script>
