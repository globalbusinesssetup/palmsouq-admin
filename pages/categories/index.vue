<template>
  <list-page
    ref="listPage"
    list-api="getCategories"
    delete-api="deleteCategory"
    route-name="categories"
    empty-store-variable="allCategories"
    :name="$t('category.cat')"
    gate="category"
    :order-options="orderOptions"
    class="tree-wrapper"
    @delete-bulk="deleteBulk"
    @list="itemList = $event"
  >
    <template
      v-slot:table="{list}"
    >
      <ul class="table-tree">
        <tree-node
          v-for="value in list"
          :node="value"
          :key="value.id"
          @edit="editNode"
          @delete="deleteNode"
        >
          <template
            v-slot:checkbox="{id}"
          >
            <input type="checkbox" :value="id" v-model="cbList">
          </template>

          <template
            v-slot:inner-checkbox="{id}"
          >
            <input type="checkbox" :value="id" v-model="cbList">
          </template>

        </tree-node>
      </ul>
    </template>
  </list-page>
</template>

<script>

  import ListPage from "~/components/partials/ListPage"
  import util from '~/mixin/util'
  import LazyImage from "~/components/LazyImage"
  import TreeNode from "~/components/TreeNode"

  export default {
    name: "categories",
    middleware: ['common-middleware', 'auth'],
    data() {
      return {
        itemList: [],
        cbList: [],
        orderOptions: {
          created_at: { title: this.$t('category.date') },
          title: { title: this.$t('index.title') },
          status: { title: this.$t('category.status') }
        }
      }
    },
    components: {
      TreeNode,
      LazyImage,
      ListPage
    },
    mixins: [util],
    computed: {
    },
    methods: {
      deleteBulk(){


        if(this.cbList?.length){
          this.$refs.listPage.deleteItem(this.cbList.join(','))
        }
      },

      editNode(node){
        this.$refs.listPage.editItem(node.id)
      },
      deleteNode(node){
        this.$refs.listPage.deleteItem(node.id)
      }
    },
    mounted() {
    }
  }
</script>

<style scoped>

</style>
