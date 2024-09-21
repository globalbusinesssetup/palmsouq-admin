<template>
  <list-page
    ref="listPage"
    list-api="getProducts"
    delete-api="deleteProduct"
    route-name="products"
    :name="$t('title.prod')"
    :order-options="orderByProduct"
    gate="product"
    @delete-bulk="deleteBulk"
    @list="itemList = $event"
  >
    <template v-slot:table="{list}">
        <tr class="lite-bold">
          <th class="w-50x mx-w-50x">
            <input type="checkbox" @change="checkAll">
          </th>
          <th>{{ $t('index.title') }}</th>
          <th>{{ $t('category.status') }}</th>
          <th>{{ $t('prod.brand') }}</th>
          <th>{{ $t('error.cat') }}</th>
          <th>{{ $t('prod.tRule') }}</th>
          <th>{{ $t('prod.purchased') }}({{ currencyIcon }})</th>
          <th>{{ $t('prod.selling') }}({{ currencyIcon }})</th>
          <th>{{ $t('prod.offered') }}({{ currencyIcon }})</th>
          <th>{{ $t('category.created') }}</th>
          <th />
          <th />
        </tr>

        <tr v-for="(value, index) in list" :key="index">
          <td class="w-50x mx-w-50x">
            <input type="checkbox" :value="value.id" v-model="cbList">
          </td>
          <td>
            <nuxt-link
              class="dply-felx j-left link"
              :to="`/products/${value.id}`"
            >
              <lazy-image
                class="mr-20"
                :data-src="getThumbImageURL(value.image)"
                :alt="value.title"
              />
              <h5 class="mx-w-400x">{{ value.title }}</h5>
            </nuxt-link>
          </td>
          <td
            class="status"
            :class="{active: value.status == 1 }"
          >
            <span>{{ getStatus(value.status) }}</span>
          </td>
          <td>
            <nuxt-link
              v-if="value.brand"
              class="link"
              :to="`brands/${value.brand.id}`"
            >
              {{ value.brand.title }}
            </nuxt-link>
            <span v-else>{{ $t('prod.na') }}</span>
          </td>

          <td>
            <span class="dply-felx f-wrap gap-10 mx-w-300x j-left">
               <nuxt-link
                 v-for="(data, index) in value.product_categories"
                 :key="index"
                 class="link"
                 :to="`/categories/${data.category.id}`"
               >
                {{ data.category.title }}
              </nuxt-link>
            </span>

          </td>

          <td>
            <nuxt-link
              v-if="value.tax_rules"
              class="link"
              :to="`tax-rules/${value.tax_rules.id}`"
            >
              {{ value.tax_rules.title }}
            </nuxt-link>
            <span v-else>{{ $t('prod.na') }}</span>
          </td>
          <td>{{ value.purchased }}</td>
          <td>{{ value.selling }}</td>
          <td>
            <span v-if="value.offered">
               {{ value.offered }}
            </span>
          </td>
          <td>{{ value.created }}</td>
          <td>
            <nuxt-link
              class="lite-btn button"
              :to="`/rating-reviews?product=${value.id}`"
            >
              {{ $t('prod.reviews') }}
            </nuxt-link>
          </td>
          <td>
            <button
              v-if="$can('product', 'edit')"
              @click.prevent="$refs.listPage.editItem(value.id)"
              class="lite-btn"
            >
              {{ $t('category.edit') }}
            </button>
            <button
              v-if="$can('product', 'delete')"
              @click.prevent="$refs.listPage.deleteItem(value.id)"
              class="delete-btn lite-btn"
            >
              {{ $t('category.delete') }}
            </button>
          </td>
        </tr>
    </template>
  </list-page>
</template>

<script>
  import ListPage from "~/components/partials/ListPage"
  import {mapGetters} from 'vuex'
  import util from '~/mixin/util'
  import LazyImage from "~/components/LazyImage"
  import bulkDelete from "~/mixin/bulkDelete";

  export default {
    name: "tax-rule",
    middleware: ['common-middleware', 'auth'],
    data(){
      return {
        orderByProduct: {
          title: { title: this.$t('index.title') },
          category_id: { title: this.$t('category.catUp') },
          purchased: { title: this.$t('prod.purchased') },
          selling: { title: this.$t('prod.selling') },
          offered: { title: this.$t('prod.offered') },
          created_at: { title: this.$t('category.date') },
          status: { title: this.$t('category.status') }
        }
      }
    },
    mixins: [util, bulkDelete],
    components: {
      LazyImage,
      ListPage
    },
    computed: {
      currencyIcon() {
        return this.setting?.currency_icon || '$'
      },
      ...mapGetters('setting', ['setting'])
    },
    methods:{
    },
    mounted() {
    }
  }
</script>

<style scoped>

</style>
