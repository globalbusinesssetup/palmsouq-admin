<template>
  <data-page
    ref="dataPage"
    set-api="setTestimonial"
    get-api="getTestimonial"
    set-image-api="uploadSiteFeatureImage"
    route-name="testimonials"
    :name="$t('title.tm')"
    :validation-keys="['testimonial']"
    :file-keys="['id', 'testimonial']"
    :result="result"
    gate="home_slider"
    @result="result = $event"
    :hideImage="true"
  >
    <template v-slot:form="{ hasError }">
      <div v-if="loading" class="spinner-wrapper">
        <spinner :radius="70" color="primary" />
      </div>

      <div class="input-wrapper">
        <label>{{ $t("title.client") }}</label>
        <input
          type="text"
          :placeholder="$t('title.client')"
          @change="slugChange"
          v-model="result.client_name"
          :class="{ invalid: !result.client_name && hasError }"
        />
        <span class="error" v-if="!result.client_name && hasError">
          {{ $t("category.req", { type: $t("title.client") }) }}
        </span>
      </div>

      <div class="input-wrapper">
        <label>{{ $t("list.tm") }}</label>
        <textarea
          type="text"
          :placeholder="$t('list.tm')"
          @change="slugChange"
          v-model="result.testimonial"
          :class="{ invalid: !result.testimonial && hasError }"
        ></textarea>
        <span class="error" v-if="!result.testimonial && hasError">
          {{ $t("category.req", { type: $t("list.tm") }) }}
        </span>
      </div>

      <div class="input-wrapper">
        <div class="dply-felx j-left mb-20 mb-sm-15">
          <span class="mr-15">
            {{ $t("fSale.rating") }}
          </span>
          <dropdown
            :selectedKey="`${result.rating}`"
            :options="ratingObj"
            @clicked="ratingDropdownSelected"
          />
        </div>
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
import Dropdown from "~/components/Dropdown";
import { mapGetters, mapActions } from "vuex";
import WYSIWYGEditor from "../../components/WYSIWYGEditor";
import Spinner from "../../components/Spinner";

export default {
  name: "features",
  middleware: ["common-middleware", "auth"],
  data() {
    return {
      loading: false,
      result: {
        id: "",
        rating: 0,
        client_name: "",
        status: "",
        testimonial: "",
      },
      ratingObj: this.createRatingObj(),
    };
  },
  mixins: [util],
  components: {
    Spinner,
    WYSIWYGEditor,
    Dropdown,
    DataPage,
  },
  computed: {
    ...mapGetters("language", ["currentLanguage"]),
  },
  methods: {
    createRatingObj() {
      const statusObj = {};
      for (let i = 1; i <= 5; i++) {
        statusObj[i] = { title: `${i}` }; // Set both the key and the title to the same value
      }
      return statusObj; // Return the created object
    },
    editorDescriptionFile({
      deleted,
      file,
      Editor,
      cursorLocation,
      resetUploader,
    }) {
      this.editorFile({ deleted, file, Editor, cursorLocation, resetUploader });
    },
    async editorFile({ deleted, file, Editor, cursorLocation, resetUploader }) {
      if (!deleted) {
        this.loading = true;
        try {
          const fd = new FormData();
          if (!this.result.id) {
            fd.append("site_feature", JSON.stringify(this.result));
          } else {
            fd.append("detail", this.result.detail);
            fd.append("site_feature_id", this.result.id);
          }
          fd.append("photo", file);
          const data = await this.setRequest({
            params: fd,
            api: "setFeatureWysiwygImage",
          });

          if (data) {
            if (!this.result.id) {
              await this.$router.push({
                path: `/testimonials/${data.site_feature_id}`,
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
            api: "deleteFeatureWysiwygImage",
          });
        } catch (e) {
          return this.$nuxt.error(e);
        }
        this.loading = false;
      }
    },
    ratingDropdownSelected(data) {
      this.result.rating = data.key;
    },
    dropdownSelected(data) {
      this.result.status = data.key;
    },
    ...mapActions("common", ["setRequest", "deleteData"]),
  },
  async mounted() {},
};
</script>

<style scoped></style>
