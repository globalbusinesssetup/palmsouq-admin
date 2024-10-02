<template>
  <div class="product-form">
    <div v-if="loading" class="spinner-wrapper">
      <spinner :radius="60" color="primary" class="mr-15" />
    </div>

    <div class="right-area">
      <div class="sticky">
        <div class="tab-sidebar mb-20 mb-lg mb-md-15">
          <h4 class="title">
            <span class="mr-5">{{ $t("prod.pVideo") }}</span>
            <span class="fw-400 f-8">{{ $t("prod.maxVid") }}</span>
          </h4>

          <div
            v-if="$can('product', 'edit') || $can('product', 'create')"
            class="form-wrapper"
          >
            <error-formatter type="video" />

            <video-input
              v-if="mediaStorageData.URL === mediaStorage"
              :saving="videoUploading"
              :video="result.video"
              :video-thumb="result.video_thumb"
              @image-change="uploadVideo(null, $event)"
            />

            <video-upload
              v-else
              class="upload-block"
              :video="result.video"
              type="video"
              :file-uploading="videoUploading"
              @file-upload="uploadVideo"
            />
          </div>
          <div v-else-if="$can('product', 'view')" class="ptb-15 plr-20">
            <video v-if="result.video" controls autoplay>
              <source :src="getVideoURL(result.video)" type="video/mp4" />
            </video>
            <div v-else>
              {{ $t("prod.noVid") }}
            </div>
          </div>
          <video
            v-else-if="result.video && $can('product', 'view')"
            controls
            autoplay
          >
            <source :src="getVideoURL(result.video)" type="video/mp4" />
          </video>
        </div>

        <div class="tab-sidebar mb-20 mlr-lg-15 mlr-md mb-md-15">
          <h4 class="title">
            <span class="mr-5">{{ $t("prod.pImg") }}</span>
            <span class="fw-400 f-8">{{ $t("prod.suggImg") }}</span>
          </h4>

          <div v-if="!loading">
            <div
              v-if="$can('product', 'edit') || $can('product', 'create')"
              class="form-wrapper upload-block"
            >
              <error-formatter type="image" />

              <image-input
                v-if="mediaStorageData.URL === mediaStorage"
                :saving="fileUploading"
                :image="result.image"
                @image-change="imageInputChanged"
              />
              <file-upload
                v-else
                class="upload-block"
                :image="result.image"
                :file-uploading="fileUploading"
                @file-upload="uploadFile"
              />
            </div>

            <img
              v-else-if="$can('product', 'view')"
              class="mx-w-300x"
              :src="getImageURL(result.image)"
            />
          </div>
        </div>

        <div class="tab-sidebar mb-md-15" v-if="!isAdding">
          <h4 class="title">
            <span class="mr-5">{{ $t("prod.pImgs") }}</span>
            <span class="fw-400 f-8">{{ $t("prod.suggImg") }}</span>
          </h4>

          <product-images
            :product-images="result.product_images"
            @result="productImages"
          />
        </div>
      </div>
    </div>

    <div ref="productForm" class="left-area">
      <div class="tab-sidebar">
        <div>
          <div class="dply-felx gap-15 title ptb-5 b-0">
            <h4>{{ $t("prod.pForm") }}</h4>

            <button
              class="btn-clear dply-felx"
              @click.prevent="productFormOpen = !productFormOpen"
            >
              <i
                class="icon black ignore-click"
                :class="`${productFormOpen ? 'arrow-up' : 'arrow-down'}`"
              />
            </button>
          </div>

          <form
            v-if="productFormOpen"
            class="form-wrapper b-t"
            @submit.prevent="checkForm"
            :class="{ 'has-error': hasError }"
          >
            <error-formatter />

            <div class="input-wrapper">
              <label>{{ $t("index.title") }}</label>
              <input
                type="text"
                :placeholder="$t('index.title')"
                @change="slugChange"
                v-model="result.title"
                :class="{ invalid: !result.title && hasError }"
              />
              <span class="error" v-if="!result.title && hasError">
                {{ $t("category.req", { type: $t("index.title") }) }}
              </span>
            </div>

            <div class="input-wrapper">
              <label>{{ $t("category.slug") }}</label>
              <input
                type="text"
                :placeholder="$t('category.slug')"
                name="slug"
                v-model="result.slug"
                ref="slug"
                :class="{ invalid: !!!result.slug && hasError }"
              />
              <span class="error" v-if="!!!result.slug && hasError">
                {{ $t("category.req", { type: $t("category.slug") }) }}
              </span>
            </div>

            <div
              v-for="(flash, index) in result.flash_sale_product"
              :key="index"
              class="info-msg mb-20 mb-sm-15"
            >
              <p>
                {{ $t("prod.flash") }}
                (<nuxt-link
                  class="link"
                  :to="`/flash-sales/${flash.flash_sale.id}`"
                >
                  {{ flash.flash_sale.title }} </nuxt-link
                >)
              </p>
              <p>
                {{ $t("prod.flashPrice") }}:
                <b>{{ priceFormatting(flash.price) }}</b>
              </p>
            </div>

            <div class="dply-felx mlr--7-5 inputs">
              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.purchased") }}({{ currencyIcon }})</label>
                <input
                  type="number"
                  step="any"
                  :placeholder="$t('prod.purchased')"
                  v-model="result.purchased"
                  :class="{
                    invalid:
                      (result.purchased < 1 || !result.purchased) && hasError,
                  }"
                />
                <span
                  class="error"
                  v-if="(result.purchased < 1 || !result.purchased) && hasError"
                >
                  {{ $t("category.req", { type: $t("brand.price") }) }}
                </span>
              </div>

              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.selling") }}({{ currencyIcon }})</label>
                <input
                  type="number"
                  step="any"
                  :placeholder="$t('prod.selling')"
                  v-model="result.selling"
                  :class="{
                    invalid:
                      (result.selling < 1 || !result.selling) && hasError,
                  }"
                />
                <span
                  class="error"
                  v-if="(result.selling < 1 || !result.selling) && hasError"
                >
                  {{ $t("category.req", { type: $t("brand.price") }) }}
                </span>
              </div>

              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.offered") }}({{ currencyIcon }})</label>
                <input
                  type="number"
                  step="any"
                  :placeholder="$t('prod.offered')"
                  v-model="result.offered"
                />
              </div>
            </div>
            <!--dply-felx inputs-->

            <div class="dply-felx mlr--7-5 inputs">
              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.barcode") }}</label>
                <input
                  type="text"
                  step="any"
                  :placeholder="$t('prod.barcode')"
                  v-model="result.barcode"
                  :class="{
                    invalid:
                      (result.barcode < 1 || !result.barcode) && hasError,
                  }"
                />
                <span
                  class="error"
                  v-if="(result.barcode < 1 || !result.barcode) && hasError"
                >
                  {{ $t("category.req", { type: $t("brand.price") }) }}
                </span>
              </div>

              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.supplier") }}</label>
                <input
                  type="text"
                  step="any"
                  :placeholder="$t('prod.supplier')"
                  v-model="result.supplier"
                  :class="{
                    invalid:
                      (result.supplier < 1 || !result.supplier) && hasError,
                  }"
                />
                <span
                  class="error"
                  v-if="(result.supplier < 1 || !result.supplier) && hasError"
                >
                  {{ $t("category.req", { type: $t("brand.price") }) }}
                </span>
              </div>

              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.sItmCode") }}</label>
                <input
                  type="text"
                  step="any"
                  :placeholder="$t('prod.sItmCode')"
                  v-model="result.supplier_item_code"
                />
              </div>
            </div>

            <div class="dply-felx inputs pos-rel w-100">
              <div class="input-wrapper">
                <label class="block">{{ $t("error.cat") }}</label>

                <div
                  data-ignore="multiple-category"
                  @click.prevent="showCategories = !showCategories"
                  class="custom-dropdown cp"
                >
                  <span class="ignore-click"
                    >{{ $t("title.choose") }} ({{
                      result.product_categories.length
                    }})
                    <i
                      class="icon black ignore-click"
                      :class="[
                        { 'arrow-up': showCategories },
                        { 'arrow-down': !showCategories },
                      ]"
                    />
                  </span>
                </div>
                <ul
                  v-if="showCategories"
                  class="multiple-wrap"
                  v-outside-click="closeDrop"
                  id="multiple-category"
                >
                  <input
                    class="mb-10"
                    type="text"
                    :placeholder="$t('list.sh')"
                    v-model="categorySearch"
                    @input="doCategorySearch"
                  />
                  <label
                    v-for="(item, index) in searchedCategories"
                    :key="index"
                    :label-for="`all-cat-${index}`"
                  >
                    <input
                      :id="`all-cat-${index}`"
                      type="checkbox"
                      v-model="result.product_categories"
                      :value="index"
                    />
                    {{ item.title }}
                  </label>
                </ul>
              </div>

              <div v-if="selectedCategories" class="input-wrapper mlr-7-5">
                <label class="block">{{ $t("title.prim") }}</label>
                <dropdown
                  class="left-dd"
                  :selectedKey="result.primary_category_id"
                  :options="selectedCategories"
                  @clicked="primaryCategorySelected"
                />
              </div>
            </div>

            <div class="dply-felx inputs mlr--7-5">
              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.unit") }}</label>
                <input
                  type="text"
                  :placeholder="$t('prod.unit')"
                  v-model="result.unit"
                  :class="{ invalid: !!!result.unit && hasError }"
                />
                <span class="error" v-if="!!!result.unit && hasError">
                  {{ $t("category.req", { type: $t("prod.unit") }) }}
                </span>
              </div>

              <div class="input-wrapper mlr-7-5">
                <label>{{ $t("prod.badge") }}</label>
                <input
                  type="text"
                  :placeholder="$t('prod.badge')"
                  v-model="result.badge"
                />
              </div>
            </div>

            <div
              class="input-wrapper"
              :class="{ 'whysigwyg-error': !!!result.overview && hasError }"
            >
              <WYSIWYGEditor
                :title="$t('prod.overview')"
                :description="result.overview"
                @change="result.overview = $event"
                @file="editorOverviewFile"
              />
              <span class="error" v-if="!!!result.overview && hasError">
                {{ $t("category.req", { type: $t("prod.overview") }) }}
              </span>
            </div>
            <div
              class="input-wrapper"
              :class="{ 'whysigwyg-error': !!!result.description && hasError }"
            >
              <WYSIWYGEditor
                :description="result.description"
                @change="result.description = $event"
                @file="editorDescriptionFile"
              />
              <span class="error" v-if="!!!result.description && hasError">
                {{ $t("category.req", { type: $t("prod.desc") }) }}
              </span>
            </div>

            <div class="dply-felx inputs mlr--7-5">
              <div class="input-wrapper mlr-7-5">
                <label class="block">{{ $t("category.status") }}</label>
                <dropdown
                  :selectedKey="`${result.status}`"
                  :options="statusObj"
                  @clicked="dropdownSelected"
                />
              </div>

              <div class="input-wrapper mlr-7-5">
                <label class="block">{{ $t("prod.brand") }}</label>
                <dropdown
                  v-if="allBrands"
                  :default-null="true"
                  :selectedKey="result.brand_id"
                  :options="allBrands"
                  @clicked="brandSelected"
                />
              </div>
            </div>

            <div class="input-wrapper">
              <label>{{ $t("prod.tags") }}</label>
              <tag-search
                @add="addTag"
                @delete="deleteTag"
                :tags="result.tags"
              />
            </div>

            <div class="f-wrap dply-felx inputs mlr--7-5">
              <div class="input-wrapper mlr-7-5">
                <label class="block">{{ $t("prod.tRule") }}</label>
                <dropdown
                  v-if="allTaxRules"
                  :selectedKey="result.tax_rule_id"
                  :options="allTaxRules"
                  @clicked="taxRuleSelected"
                />

                <span class="error" v-if="!!!result.tax_rule_id && hasError">
                  {{ $t("category.req", { type: $t("brand.tRule") }) }}
                </span>
              </div>

              <div class="input-wrapper mlr-7-5">
                <label class="block">{{ $t("dataPage.shipRule") }}</label>
                <dropdown
                  v-if="allShippingRules"
                  :selectedKey="result.shipping_rule_id"
                  :options="allShippingRules"
                  @clicked="shippingRuleSelected"
                />
                <span
                  class="error"
                  v-if="!!!result.shipping_rule_id && hasError"
                >
                  {{ $t("category.req", { type: $t("brand.shipRule") }) }}
                </span>
              </div>

              <div class="input-wrapper mlr-7-5">
                <label class="block">{{ $t("profile.dleDeal") }}</label>
                <dropdown
                  v-if="allBundleDeals"
                  :default-null="true"
                  :selectedKey="result.bundle_deal_id"
                  :options="allBundleDeals"
                  @clicked="bundleDealSelected"
                />
              </div>
            </div>

            <div class="mb-10 mb-sm-5 cb-wrapper">
              <span class="block mb-10">{{ $t("dataPage.prodCol") }}</span>
              <label
                v-for="(value, index) in allProductCollections"
                :key="index"
                class="mr-15 mb-10"
              >
                <input
                  type="checkbox"
                  :value="value.id"
                  v-model="result.product_collections"
                />
                {{ value.title }}
              </label>
            </div>

            <div class="mb-20 mb-sm-15 cb-wrapper">
              <span class="block mb-10">
                {{ $t("prod.pInfo") }}
              </span>
              <p>
                <label class="mr-15">
                  {{ $t("prod.refund") }}
                </label>

                <label>
                  <input
                    type="radio"
                    name="refundable"
                    value="1"
                    v-model="result.refundable"
                  />
                  {{ $t("prod.yes") }}
                </label>

                <label>
                  <input
                    type="radio"
                    name="refundable"
                    value="2"
                    v-model="result.refundable"
                  />
                  {{ $t("prod.no") }}
                </label>
              </p>

              <p>
                <label class="mr-15">
                  {{ $t("prod.warranty") }}
                </label>

                <label>
                  <input
                    type="radio"
                    name="warranty"
                    value="1"
                    v-model="result.warranty"
                  />
                  {{ $t("prod.yes") }}
                </label>

                <label>
                  <input
                    type="radio"
                    name="warranty"
                    value="2"
                    v-model="result.warranty"
                  />
                  {{ $t("prod.no") }}
                </label>
              </p>
            </div>

            <div class="input-wrapper">
              <label>{{ $t("category.mTitle") }}</label>
              <input
                type="text"
                :placeholder="$t('category.mTitle')"
                v-model="result.meta_title"
                :class="{ invalid: !!!result.meta_title && hasError }"
              />
              <span class="error" v-if="!!!result.meta_title && hasError">
                {{ $t("category.req", { type: $t("category.mTitle") }) }}
              </span>
            </div>
            <div class="input-wrapper">
              <label>{{ $t("ship.mk") }} ({{ $t("ship.csk") }})</label>
              <textarea
                :placeholder="$t('ship.mk')"
                v-model="result.meta_keywords"
                :class="{ invalid: !!!result.meta_keywords && hasError }"
              />
              <span class="error" v-if="!!!result.meta_keywords && hasError">
                {{ $t("category.req", { type: $t("ship.mk") }) }}
              </span>
            </div>

            <div class="input-wrapper">
              <label>{{ $t("category.mDesc") }}</label>
              <textarea
                :placeholder="$t('category.mDesc')"
                v-model="result.meta_description"
                :class="{ invalid: !!!result.meta_description && hasError }"
              />
              <span class="error" v-if="!!!result.meta_description && hasError">
                {{ $t("category.req", { type: $t("category.mDesc") }) }}
              </span>
            </div>

            <div v-if="!loading" style="padding-bottom: 20px">
              <h4 class="title" style="padding-left: 0; border-bottom: 0">
                Banner
              </h4>
              <div v-if="$can('product', 'edit') || $can('product', 'create')">
                <error-formatter type="image" />

                <image-input
                  v-if="mediaStorageData.URL === mediaStorage"
                  :saving="fileUploading"
                  :image="result.image"
                  @image-change="imageInputChanged"
                />
                <file-upload
                  v-else
                  class="upload-block"
                  :image="result.image"
                  :file-uploading="fileUploading"
                  @file-upload="uploadFile"
                  :imageContainerStyle="{ height: '300px' }"
                />
              </div>

              <img
                v-else-if="$can('product', 'view')"
                class="mx-w-300x"
                :src="getImageURL(result.image)"
              />
            </div>

            <div
              v-if="
                validLicence &&
                ($can('product', 'edit') || $can('product', 'create'))
              "
              class="dply-felx j-right gap-15"
            >
              <ajax-button
                name="save-edit"
                class="primary-btn"
                :text="$t('list.svn')"
                :fetching-data="formSubmitting && !redirect"
              />
              <ajax-button
                name="save"
                class="primary-btn"
                :text="$t('setting.sv')"
                :fetching-data="formSubmitting && redirect"
              />
            </div>
          </form>
        </div>
      </div>

      <div class="tab-sidebar mt-15" v-if="!isAdding" ref="productInventory">
        <product-inventory
          v-if="currentPrice"
          :attributes="allAttributes"
          :product-price="parseFloat(currentPrice)"
          @has-error="scrollToTop('productInventory')"
        />
      </div>
    </div>
    <!--left-area-->
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
import ErrorFormatter from "~/components/ErrorFormatter";
import WYSIWYGEditor from "~/components/WYSIWYGEditor";
import ProductImages from "~/components/partials/ProductImages";
import FileUpload from "~/components/FileUpload";
import VideoUpload from "~/components/VideoUpload";
import TagSearch from "~/components/TagSearch";
import AjaxButton from "~/components/AjaxButton";
import ProductInventory from "~/components/partials/ProductInventory";
import DataPage from "~/components/partials/DataPage";
import Dropdown from "~/components/Dropdown";
import Spinner from "~/components/Spinner";
import util from "~/mixin/util";
import ImageInput from "~/components/ImageInput";
import VideoInput from "~/components/VideoInput";
import outsideClick from "~/directives/outside-click.js";
import { debounce } from "debounce";

export default {
  name: "product",
  middleware: ["common-middleware", "auth"],
  data() {
    return {
      licence: null,
      categorySearch: "",

      productFormOpen: true,
      showCategories: false,
      validLicence: false,
      routeName: "products",
      getApi: "getProduct",
      setApi: "setProduct",
      setImageApi: "setProductImage",
      setVideoApi: "setProductVideo",
      fileKeys: ["id", "tax_rule_id", "shipping_rule_id"],
      validationKeys: [
        "title",
        "slug",
        "unit",
        "meta_title",
        "meta_description",
        "description",
        "overview",
        "selling",
        "purchased",
      ],
      result: {
        id: "",
        title: "",
        tags: ",",
        overview: "",
        description: "",
        status: "2",
        brand_id: "",
        primary_category_id: "",
        category_id: "",
        bundle_deal_id: "",
        unit: "",
        badge: "",
        subcategory_id: "",
        tax_rule_id: "",
        shipping_rule_id: "",
        purchased: "",
        selling: "",
        offered: "",
        barcode: "",
        supplier: "",
        supplier_item_code: "",
        refundable: 1,
        warranty: 1,
        meta_description: "",
        meta_keywords: "",
        flash_sale_product: "",
        meta_title: "",
        image: "",
        slug: "",
        video: "",
        product_images: [],
        product_collections: [],
        product_categories: [],
      },
      hasError: false,
      loading: false,
      formSubmitting: false,
      redirect: false,
      fileUploading: false,
      videoUploading: false,
      searchedCategories: {},
    };
  },
  directives: {
    outsideClick,
  },
  mixins: [util],
  components: {
    VideoInput,
    ImageInput,
    DataPage,
    Dropdown,
    WYSIWYGEditor,
    ProductImages,
    FileUpload,
    VideoUpload,
    TagSearch,
    AjaxButton,
    ProductInventory,
    ErrorFormatter,
    Spinner,
  },
  computed: {
    productCategories() {
      return this.result.product_categories;
    },
    selectedCategories() {
      let sc = null;
      if (this.allCategories && this.productCategories.length) {
        Object.keys(this.allCategories).forEach((i) => {
          if (this.productCategories.includes(i)) {
            sc = { ...sc, ...{ [i]: this.allCategories[i] } };
          }
        });
      }
      return sc;
    },
    currencyPosition() {
      return this.setting?.currency_position;
    },
    currentPrice() {
      return this.result.offered > 0
        ? this.result.offered
        : this.result.selling > 0
        ? this.result.selling
        : 0;
    },
    id() {
      return !this.isAdding ? this.$route?.params?.id : "";
    },
    isAdding() {
      return isNaN(this.$route?.params?.id);
    },
    currencyIcon() {
      return this.setting?.currency_icon || "$";
    },
    ...mapGetters(["mediaStorage"]),
    ...mapGetters("admin", ["publicKey"]),
    ...mapGetters("language", ["currentLanguage"]),
    ...mapGetters("setting", ["setting"]),
    ...mapGetters("common", [
      "allCategories",
      "allTaxRules",
      "allAttributes",
      "allBrands",
      "allProductCollections",
      "allBundleDeals",
      "allShippingRules",
    ]),
  },
  methods: {
    doCategorySearch: debounce(function () {
      this.searchOnCategory();
    }, 500),
    searchOnCategory() {
      this.searchedCategories = {};

      Object.keys(this.allCategories)
        .filter((element) =>
          this.allCategories[element].title
            .toLowerCase()
            .includes(this.categorySearch.toLowerCase())
        )
        .forEach((i) => {
          this.searchedCategories[i] = this.allCategories[i];
        });

      if (
        !Object.keys(this.searchedCategories)?.length &&
        !this.categorySearch
      ) {
        this.searchedCategories = this.allCategories;
      }
    },
    closeDrop() {
      this.showCategories = false;
    },
    productImages(evt) {
      this.result.product_images = [];

      this.$nextTick(() => {
        this.result.product_images = evt;
      });
    },
    imageInputChanged(evt) {
      this.uploadFile(null, evt);
    },
    addTag(tag) {
      if (!this.result.tags) {
        this.result.tags = ",";
      }
      this.result.tags = `${this.result.tags}${tag},`;
    },
    deleteTag(tag) {
      this.result.tags = this.result.tags.replace(`${tag},`, "");
    },
    dropdownSelected(data) {
      this.result.status = data.key;
    },
    primaryCategorySelected(data) {
      this.result.primary_category_id = data.key;
    },
    categorySelected(data) {
      this.result.category_id = data.key;

      if (this.selectedSubCategories) {
        this.result.subcategory_id = Object.keys(this.selectedSubCategories)[0];
      } else {
        this.result.subcategory_id = "";
      }
    },
    taxRuleSelected(data) {
      this.result.tax_rule_id = data.key;
    },
    shippingRuleSelected(data) {
      this.result.shipping_rule_id = data.key;
    },
    brandSelected(data) {
      this.result.brand_id = data.key;
    },
    bundleDealSelected(data) {
      this.result.bundle_deal_id = data.key;
    },
    redirectingEnable(buttonType) {
      this.redirect = buttonType === "save";
    },
    async checkForm() {
      if (
        this.validationKeys.findIndex((i) => {
          return !this.result[i];
        }) > -1
      ) {
        this.hasError = true;
        return false;
      }
      this.redirectingEnable(event.submitter.name);
      this.formSubmitting = true;
      try {
        delete this.result.created_at;
        delete this.result.updated_at;
        const data = await this.setById({
          id: this.id,
          params: this.result,
          api: this.setApi,
        });

        if (data) {
          this.result = Object.assign({}, data);

          this.result.product_collections = [
            ...new Set(
              this.result?.product_collections?.map((o) => {
                return o.product_collection_id;
              })
            ),
          ];
          this.result.product_categories = [
            ...new Set(
              this.result?.product_categories?.map((o) => {
                return o.category_id.toString();
              })
            ),
          ];

          this.$router.push({
            path: `/${this.routeName}${
              this.redirect ? "" : "/" + this.result.id
            }`,
          });
        } else {
          this.scrollToTop();
        }
      } catch (e) {
        return this.$nuxt.error(e);
      }
      this.formSubmitting = false;
    },

    scrollToTop(ref = "productForm") {
      this.$refs[ref].scrollIntoView({ behavior: "smooth" });
    },
    async fetchingData() {
      try {
        this.loading = true;
        this.result = Object.assign(
          {},
          await this.getById({ id: this.id, params: {}, api: this.getApi })
        );

        this.result.product_collections = [
          ...new Set(
            this.result?.product_collections?.map((o) => {
              return o.product_collection_id;
            })
          ),
        ];
        this.result.product_categories = [
          ...new Set(
            this.result?.product_categories?.map((o) => {
              return o.category_id.toString();
            })
          ),
        ];

        this.loading = false;
      } catch (e) {
        return this.$nuxt.error(e);
      }
    },
    async uploadVideo(file, video = null) {
      this.videoUploading = true;
      try {
        let params = {};
        if (file) {
          const fd = new FormData();

          this.fileKeys.forEach((i) => {
            fd.append(i, this.result[i]);
          });
          fd.append("video_file", file.video);
          fd.append("thumb", file.thumb);

          params = fd;
        } else {
          this.fileKeys.forEach((i) => {
            params[i] = this.result[i];
          });

          params["video_file"] = video.video;
          params["thumb"] = video.thumb;
        }
        const data = await this.setImageById({
          id: this.id,
          params: params,
          api: this.setVideoApi,
        });

        if (data) {
          this.result = Object.assign({}, data);
          this.result.product_collections = [
            ...new Set(
              this.result?.product_collections?.map((o) => {
                return o.product_collection_id;
              })
            ),
          ];
          this.result.product_categories = [
            ...new Set(
              this.result?.product_categories?.map((o) => {
                return o.category_id.toString();
              })
            ),
          ];

          await this.$router.push({
            path: `/${this.routeName}/${this.result.id}`,
          });
        }
      } catch (e) {
        return this.$nuxt.error(e);
      }
      this.videoUploading = false;
    },
    async uploadFile(file, name = null) {
      this.fileUploading = true;
      try {
        let params = {};
        if (file) {
          const fd = new FormData();
          this.fileKeys.forEach((i) => {
            fd.append(i, this.result[i]);
          });
          fd.append("photo", file);
          params = fd;
        } else {
          this.fileKeys.forEach((i) => {
            params[i] = this.result[i];
          });
          params["photo"] = name;
        }

        const data = await this.setImageById({
          id: this.id,
          params: params,
          api: this.setImageApi,
        });

        if (data) {
          this.result = Object.assign({}, data);
          this.result.product_collections = [
            ...new Set(
              this.result?.product_collections?.map((o) => {
                return o.product_collection_id;
              })
            ),
          ];
          this.result.product_categories = [
            ...new Set(
              this.result?.product_categories?.map((o) => {
                return o.category_id.toString();
              })
            ),
          ];

          await this.$router.push({
            path: `/${this.routeName}/${this.result.id}`,
          });
        }
      } catch (e) {
        return this.$nuxt.error(e);
      }
      this.fileUploading = false;
    },
    editorDescriptionFile({
      deleted,
      file,
      Editor,
      cursorLocation,
      resetUploader,
    }) {
      this.editorFile(
        { deleted, file, Editor, cursorLocation, resetUploader },
        this.wysiwygType.PRODUCT_DESCRIPTION
      );
    },
    editorOverviewFile({
      deleted,
      file,
      Editor,
      cursorLocation,
      resetUploader,
    }) {
      this.editorFile(
        { deleted, file, Editor, cursorLocation, resetUploader },
        this.wysiwygType.PRODUCT_OVERVIEW
      );
    },
    async editorFile(
      { deleted, file, Editor, cursorLocation, resetUploader },
      type
    ) {
      if (!deleted) {
        this.loading = true;
        try {
          const fd = new FormData();
          if (!this.result.id) {
            fd.append("product", JSON.stringify(this.result));
          } else {
            fd.append("overview", this.result.overview);
            fd.append("description", this.result.description);
            fd.append("item_id", this.result.id);
          }
          fd.append("type", type);
          fd.append("photo", file);

          const data = await this.setWysiwygImage(fd);
          if (data) {
            if (!this.result.id) {
              await this.$router.push({
                path: `/${this.routeName}/${data.item_id}`,
              });
            } else {
              Editor.insertEmbed(cursorLocation, "image", data.url);
              resetUploader();
            }
          }
        } catch (e) {
          return this.$nuxt.error(e);
        }
        this.loading = false;
      } else {
        this.loading = true;
        try {
          await this.deleteData({
            params: this.getImageName(file),
            api: "deleteWysiwygImage",
          });
        } catch (e) {
          return this.$nuxt.error(e);
        }
        this.loading = false;
      }
    },
    ...mapActions("common", [
      "getById",
      "setById",
      "setImageById",
      "getDropdownList",
      "setWysiwygImage",
      "deleteData",
    ]),
  },
  async mounted() {
    const domain = window.location.hostname;

    if (
      domain.includes("admin.ishop.cholobangla.com") ||
      domain.includes("localhost") ||
      domain.includes("127.0.0.1")
    ) {
      this.validLicence = true;
    } else if (this.publicKey) {
      this.licence = this.phpDecryption(this.publicKey);

      const licenceObj = JSON.parse(this.licence);
      const validLicenceItem =
        !licenceObj?.p || (licenceObj?.p && licenceObj?.p === "ishop");

      if (this.licence.includes(domain) && validLicenceItem) {
        this.validLicence = true;
      }
    }

    if (!this.isAdding) {
      await this.fetchingData();
    }
    if (
      !this.allCategories ||
      !this.allTaxRules ||
      !this.allAttributes ||
      !this.allBrands ||
      !this.allProductCollections ||
      !this.allBundleDeals ||
      !this.allShippingRules
    ) {
      this.loading = true;
      try {
        await this.getDropdownList();
      } catch (e) {
        return this.$nuxt.error(e);
      }
      this.loading = false;
    }

    this.searchedCategories = this.allCategories;
  },
};
</script>

<style scoped></style>
