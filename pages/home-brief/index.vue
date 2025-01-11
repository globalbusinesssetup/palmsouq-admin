<template>
  <data-page
    ref="dataPage"
    set-api="setHomeBrief"
    get-api="getHomeBrief"
    set-image-api="setHomeBriefImage"
    route-name="home-brief"
    :name="$t('homeBrief.homeUp')"
    gate="category"
    :validation-keys="['title', 'subtitle', 'meta_description']"
    :file-keys="['id', 'status']"
    :result="result"
    @result="resultData"
  >
    <template v-slot:form="{ hasError }">
      <div class="input-wrapper">
        <label>{{ $t("homeBrief.title") }}</label>
        <input
          type="text"
          :placeholder="$t('homeBrief.title')"
          v-model="result.title"
          ref="title"
          @change="slugChange"
          :class="{ invalid: !!!result.title && hasError }"
        />
        <span class="error" v-if="!!!result.title && hasError">
          {{ $t("category.req", { type: $t("homeBrief.title") }) }}
        </span>
      </div>

      <div class="input-wrapper">
        <label>{{ $t("homeBrief.subTitle") }}</label>
        <input
          type="text"
          :placeholder="$t('homeBrief.subTitle')"
          v-model="result.subtitle"
          :class="{ invalid: !!!result.subtitle && hasError }"
        />
        <span class="error" v-if="!!!result.title && hasError">
          {{ $t("category.req", { type: $t("homeBrief.subTitle") }) }}
        </span>
      </div>
      <div class="input-wrapper">
        <label>{{ $t("homeBrief.desc") }}</label>
        <textarea
          :placeholder="$t('homeBrief.desc')"
          v-model="result.description"
          :class="{ invalid: !!!result.description && hasError }"
        />
        <span class="error" v-if="!!!result.description && hasError">
          {{ $t("category.req", { type: $t("homeBrief.desc") }) }}
        </span>
      </div>

      <div class="input-wrapper">
        <div class="dply-felx j-left mb-20 mb-sm-15">
          <span class="mr-15">
            {{ $t("category.status") }}
          </span>

          <dropdown
            :selectedKey="`${result.status}`"
            :options="statusObj"
            @clicked="dropdownSelected"
          />
        </div>
      </div>
    </template>
  </data-page>
</template>

<script>
import DataPage from "~/components/partials/DataPage";
import util from "~/mixin/util";
import { mapGetters, mapActions } from "vuex";
import Dropdown from "../../components/Dropdown.vue";

export default {
  name: "home-brief",
  middleware: ["common-middleware", "auth"],
  data() {
    return {
      result: {
        id: "",
        title: "",
        status: 2,
        description: "",
        subtitle: "",
        image: "",
      },
      getApi: "getHomeBrief",
    };
  },
  mixins: [util],
  components: {
    DataPage,
    Dropdown
  },
  computed: {
    ...mapGetters("language", ["currentLanguage"]),
    ...mapGetters("common", ["getHomeBrief"]),
  },
  methods: {
    async fetchingData () {
      try {
        this.loading = true;
        this.result = Object.assign(
          {},
          await this.getById({ id: this.id, params: {}, api: this.getApi })
        );

        this.loading = false;
      } catch (e) {
        return this.$nuxt.error(e);
      }
    },
    ...mapActions("common", ["getById", "setHomeBrief", "setHomeBriefImage"]),
  },
  async mounted() {
      try {
        await this.fetchingData();
      } catch (e) {
        return this.$nuxt.error(e);
      }
  },
};
</script>

<style scoped></style>
