<template>
  <list-page
    ref="listPage"
    list-api="getBundleDeals"
    delete-api="deleteBundleDeal"
    route-name="bundle-deals"
    empty-store-variable="allBundleDeals"
    :name="$t('brand.dleDeal')"
    :order-options="orderOptions"
    gate="bundle_deal"
    @delete-bulk="deleteBulk"
    @list="itemList = $event"
  >
    <template v-slot:table="{list}">
        <tr class="lite-bold">
          <th class="w-50x mx-w-50x">
            <input type="checkbox" @change="checkAll">
          </th>
          <th>{{ $t('index.title') }}</th>
          <th>{{ $t('brand.buy') }}</th>
          <th>{{ $t('brand.free') }}</th>
          <th>{{ $t('category.created') }}</th>
          <th>&nbsp;</th>
        </tr>

        <tr v-for="(value, index) in list" :key="index">
          <td class="w-50x mx-w-50x">
            <input type="checkbox" :value="value.id" v-model="cbList">
          </td>
          <td>
            <nuxt-link
              :to="`/bundle-deals/${value.id}`"
              class="dply-felx j-left link"
            >
              <h5 class="mx-w-300x">{{ value.title }}</h5>
            </nuxt-link>
          </td>
          <td>{{ value.buy }}</td>
          <td>{{ value.free }}</td>
          <td>{{ value.created }}</td>
          <td>
            <button
              v-if="$can('bundle_deal', 'edit')"
              @click.prevent="$refs.listPage.editItem(value.id)" class="lite-btn">{{ $t('category.edit') }}</button>
            <button
              v-if="$can('bundle_deal', 'delete')"
              @click.prevent="$refs.listPage.deleteItem(value.id)" class="delete-btn lite-btn">{{ $t('category.delete') }}</button>
          </td>
        </tr>
    </template>
  </list-page>
</template>

<script>
  import bulkDelete from "~/mixin/bulkDelete";
  import util from "~/mixin/util";
  import ListPage from "~/components/partials/ListPage";

  export default {
    name: "tax-rule",
    middleware: ['common-middleware', 'auth'],
    data(){
      return {
        orderOptions:{
          title: { title: this.$t('index.title') },
          created_at: { title: this.$t('category.date') }
        }
      }
    },
    mixins: [util, bulkDelete],
    components: {
      ListPage
    },
    computed: {
    },
    methods:{
    },
    mounted() {
    }
  }
</script>

<style scoped>

</style>
