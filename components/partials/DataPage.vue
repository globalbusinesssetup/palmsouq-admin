<template>
  <div ref="formWrapper" class="tab-sidebar">
    <h4 class="title">{{ $t("profile.form", { name: name }) }}</h4>

    <div class="form-wrapper">
      <error-formatter />
      <error-formatter type="image" />
      <div v-if="loading" class="spinner-wrapper">
        <spinner :radius="60" color="primary" class="mr-15" />
      </div>

      <div v-if="!loading && !hideImage">
        <div
          v-if="
            fileKeys.length &&
            (!gate || $can(gate, 'edit') || $can(gate, 'create'))
          "
        >
          <image-input
            v-if="mediaStorageData.URL === mediaStorage"
            class="mb-20 mb-sm-15"
            :saving="fileUploading"
            :image="result.image"
            @image-change="uploadFile(null, $event)"
          />
          <file-upload
            v-else
            :image="result.image"
            :file-uploading="fileUploading"
            only-icon="upload-icon"
            class="mb-20 mb-sm-15"
            @file-upload="uploadFile"
            :imageWrapStyle="{ width: '350px' }"
          />
        </div>

        <img
          v-else-if="fileKeys.length && $can(gate, 'view')"
          class="mx-w-1920x"
          :src="getImageURL(result.image)"
        />
      </div>

      <form @submit.prevent="formSubmit" :class="{ 'has-error': hasError }">
        <slot name="form" v-bind:hasError="hasError" />
        <div v-if="!loading && showBanner">
          <h4 class="title" style="padding-left: 0; border-bottom: 0">
            Banner
          </h4>
          <div
            v-if="
              fileKeys.length &&
              (!gate || $can(gate, 'edit') || $can(gate, 'create'))
            "
          >
            <image-input
              v-if="mediaStorageData.URL === mediaStorage"
              class="mb-20 mb-sm-15"
              :saving="fileUploading"
              :image="result?.banner_image"
              @image-change="uploadFile(null, $event, null, 'banner')"
            />
            <file-upload
              v-else
              :image="result?.banner_image"
              :file-uploading="fileUploading"
              only-icon="upload-icon"
              class="mb-20 mb-sm-15"
              @file-upload="uploadFile($event, null, 'banner')"
              :imageWrapStyle="{ flex: 1 }"
            />
          </div>

          <img
            v-else-if="fileKeys.length && $can(gate, 'view')"
            class="mx-w-1920x"
            :src="getImageURL(result.image)"
          />
        </div>

        <div
          v-if="!gate || $can(gate, 'edit') || $can(gate, 'create')"
          class="dply-felx j-right single-btn"
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
</template>

<script>
import AjaxButton from "~/components/AjaxButton";
import util from "~/mixin/util";
import FileUpload from "~/components/FileUpload";
import Spinner from "~/components/Spinner";
import Dropdown from "~/components/Dropdown";
import ErrorFormatter from "~/components/ErrorFormatter";
import { mapGetters, mapActions } from "vuex";
import ImageInput from "../ImageInput";

export default {
  name: "DataPage",
  middleware: ["common-middleware", "auth"],
  data() {
    return {
      loading: false,
      redirect: false,
      formSubmitting: false,
      fileUploading: false,
      hasError: false,
    };
  },
  props: {
    result: {
      type: Object,
      default() {
        return null;
      },
    },
    emitBeforeSubmit: {
      type: Boolean,
      default: false,
    },
    name: {
      type: String,
      default: "",
    },
    gate: {
      type: String,
      default: null,
    },
    hash: {
      type: String,
      default: "",
    },
    setApi: {
      type: String,
      default: "",
    },
    setImageApi: {
      type: String,
      default: "",
    },
    getApi: {
      type: String,
      default: "",
    },
    emptyStoreVariable: {
      type: String,
      default: null,
    },
    routeName: {
      type: String,
      default: "",
    },
    validationKeys: {
      type: Array,
      default: [],
    },
    fileKeys: {
      type: Array,
      default() {
        return [];
      },
    },
    hideImage: {
      type: Boolean,
      default: false,
    },
    showBanner: {
      type: Boolean,
      default: false,
    },
  },
  mixins: [util],
  components: {
    ImageInput,
    Spinner,
    AjaxButton,
    ErrorFormatter,
    FileUpload,
    Dropdown,
  },
  computed: {
    id() {
      return !this.isAdding ? this.$route?.params?.id : "";
    },
    isAdding() {
      return isNaN(this.$route?.params?.id);
    },
    ...mapGetters(["mediaStorage"]),
  },
  methods: {
    async uploadFile(file, name = null, fieldName = "photo") {
      try {
        let params = {};
        if (file) {
          const fd = new FormData();
          this.fileKeys.forEach((i) => {
            fd.append(i, this.result[i]);
          });
          fd.append(fieldName, file);
          params = fd;
        } else {
          this.fileKeys.forEach((i) => {
            params[i] = this.result[i];
          });
          params[fieldName] = name;
        }

        this.fileUploading = true;

        const data = await this.setImageById({
          id: this.id || this.result.id,
          params: params,
          api: this.setImageApi,
        });
        console.log("image upload res: =>", data);
        if (data && (!this.gate || this.$can(this.gate, "view"))) {
          await this.$emit("result", {...this.result, image: data.image, banner_image: data.banner_image, id: data.id});
          // await this.$router.push({
          //   path: `/${this.routeName}/${this.result.id}`,
          // });
        }

        this.fileUploading = false;
        console.log(this.result);
      } catch (e) {
        return this.$nuxt.error(e);
      }
    },
    redirectingEnable(buttonType) {
      this.redirect = buttonType === "save";
    },
    formSubmit() {
      if (this.emitBeforeSubmit) {
        this.$emit("before-submit");
      } else {
        this.checkForm();
      }
    },
    async checkForm() {
      if (
        this.validationKeys.findIndex((i) => {
          return !this.result[i];
        }) > -1
      ) {
        this.hasError = true;
        console.log("checkForm",this.validationKeys);
        return false;
      }

      this.redirectingEnable(event.submitter.name);
      this.formSubmitting = true;
      try {
        delete this.result.created_at;
        delete this.result.updated_at;
        console.log("checkForm",this.result);
        console.log("id",this.id);
        const data = await this.setById({
          id: this.id || this.result.id,
          params: this.result,
          api: this.setApi,
        });
        if (data) {
          this.emptyAllList(this.emptyStoreVariable);

          await this.$emit("result", Object.assign({}, data));

          if (!this.gate || (this.$can(this.gate, "view") && this.result?.id)) {
            await this.$router.push({
              path: `/${this.routeName}${
                this.redirect ? "" : "/" + this.result?.id
              }`,
              hash: this.hash,
            });
          }
          await this.$emit("on-success", Object.assign({}, data));
        } else {
          this.$refs["formWrapper"].scrollIntoView({ behavior: "smooth" });
        }
      } catch (e) {
        return this.$nuxt.error(e);
      }
      this.formSubmitting = false;
    },
    async fetchingData() {
      try {
        this.loading = true;
        this.$emit(
          "result",
          Object.assign(
            {},
            await this.getById({
              id: this.id,
              params: { time_zone: this.timeZone },
              api: this.getApi,
            })
          )
        );
        this.loading = false;
      } catch (e) {
        return this.$nuxt.error(e);
      }
    },
    ...mapActions("common", [
      "getById",
      "setById",
      "setImageById",
      "emptyAllList",
    ]),
  },
  async mounted() {
    console.log(this.statusObj);
    if (!this.isAdding) {
      this.$nextTick(() => {
        if (!this.gate || this.$can(this.gate, "view")) {
          this.fetchingData();
        }
      });
    }
  },
};
</script>

<style scoped></style>
