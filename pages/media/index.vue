<template>
  <div>
    <h5 class="mb-20">
      {{ $t("list.show", { from: 1, to: imageCount, total: imageCount }) }}
    </h5>
    <div class="media-top">
      <div class="media-buttons">
        <!-- <ajax-button
          type="button"
          name="upload"
          class="primary-btn"
          :text="$t('Upload')"
          :fetching-data="uploading"
          @clicked="$refs.fileInput.click()"
        /> -->
        <ajax-button
          name="download"
          class="primary-btn"
          :text="$t('Download')"
        />
        <div class="dply-felx gap-10 j-left f-wrap mt-md-15">
          <table-sort class="mt-0" :order-by-options="orderByOptions" />

          <inline-pop-over :arrow="true" class="bulk-action" ref="bulkDelete">
            <template v-slot:button>
              {{ $t("title.act") }}
            </template>
            <template v-slot:content>
              <button
                :disabled="selectedImageList.length < 1"
                @click.prevent="deleteMultipleImages()"
                class="outline-btn"
              >
                {{ $t("category.delete") }}
              </button>
            </template>
          </inline-pop-over>
        </div>
      </div>

      <form class="search-input media-search">
        <input type="text" :placeholder="$t('list.sh')" v-model="search" />
        <!-- <button class="primary-btn">{{ $t("list.srch") }}</button> -->
      </form>
    </div>

    <div v-if="loading" class="spinner-wrapper">
      <spinner :radius="100" color="primary" class="mr-15" />
    </div>

    <div v-else class="media-list">
      <div class="image-container" style="padding-top: 20px">
        <div
          v-for="(data, index) in thumbs"
          :key="index"
          class="card media-card"
          :style="
            selectedImage === data
              ? { border: '1px solid #b3d1ff' }
              : { border: '1px solid transparent' }
          "
          @click="setImage(data)"
        >
          <image-card
            class="mr-20"
            :data-src="getImageURL(data)"
            :alt="thumbToMain(data)"
          />
          <p class="media-name">
            {{ thumbToMain(data) }}
          </p>
          <p
            @click.stop="setSelectedImage(index)"
            :class="selectedImageList.includes(index) ? 'check' : 'uncheck'"
          >
            {{ selectedImageList.includes(index) ? "✓" : "" }}
          </p>
          <button class="" @click.prevent="deleteImage(index)">✖</button>
        </div>
      </div>
      <!-- Right side  -->
      <div
        v-if="selectedImage"
        style="
          border-left: 1px solid #cccccc;
          padding-left: 10px;
          padding-top: 20px;
        "
      >
        <img :src="getImageURL(selectedImage)" style="width: 270px" alt="" />
        <div
          class=""
          style="
            margin-top: 20px;
            padding-top: 20px;
            border-top: 1px solid #cccccc;
          "
        >
          <div class="">
            <p style="color: black; font-weight: 600">Name</p>
            <p
              style="
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
              "
            >
              {{ thumbToMain(selectedImage).split(".")[0] }}
            </p>
          </div>
          <div class="" style="margin-top: 7px">
            <p style="color: black; font-weight: 600">Full URL</p>
            <div
              class=""
              style="
                border: 1px solid #cccccc;
                display: flex;
                align-items: center;
                padding-left: 10px;
              "
            >
              <p
                style="
                  flex: 1;
                  white-space: nowrap;
                  text-overflow: ellipsis;
                  overflow: hidden;
                "
              >
                {{ thumbToMain(selectedImage) }}
              </p>
              <button
                @click="copyText"
                style="
                  border: none;
                  border-radius: 0;
                  background-color: #cccccc;
                  display: flex;
                  align-items: center;
                  justify-content: center;
                "
              >
                <i class="icon ui"></i>
              </button>
            </div>
          </div>
          <div class="" style="margin-top: 7px">
            <p style="color: black; font-weight: 600">Alt text</p>
            <p
              style="
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
              "
            >
              {{ thumbToMain(selectedImage).split(".")[0] }}
            </p>
          </div>
        </div>
      </div>
    </div>
    <pop-over
      v-if="selectedImage"
      title="Image data"
      @close="$emit('close')"
      elem-id="media-pop-over"
      :layer="false"
      class="address-popup popup-top-auto"
    >
      <template v-slot:content>
        <div>
          <img :src="getImageURL(selectedImage)" alt="" />
        </div>
      </template>
    </pop-over>
    <input
      type="file"
      accept="image/*"
      @change="imageChanged"
      ref="fileInput"
      style="display: none"
      multiple
    />
  </div>
</template>

<script>
import util from "~/mixin/util";
import { mapActions } from "vuex";
import LazyImage from "../../components/LazyImage";
import ImageCard from "../../components/ImageCard";
import Spinner from "../../components/Spinner";
import AjaxButton from "~/components/AjaxButton";
import validation from "~/mixin/validation";
import TableSort from "~/components/partials/TableSort";
import InlinePopOver from "~/components/InlinePopOver";
import PopOver from "~/components/PopOver";

export default {
  name: "images",
  middleware: ["common-middleware", "auth"],
  data() {
    return {
      loading: false,
      imageList: [],
      selectedImageList: [],
      selectedImage: "",
      uploading: false,
      search: "",
    };
  },
  components: {
    Spinner,
    LazyImage,
  },
  mixins: [util, validation],
  computed: {
    imageCount() {
      return this.thumbs.length;
    },
    thumbs() {
      return this.imageList
        ?.filter((str) => !str.startsWith("thumb-"))
        .filter((image) => {
          const mainImage = this.thumbToMain(image);
          return mainImage.toLowerCase().includes(this.search.toLowerCase());
        });
    },
  },
  methods: {
    async imageChanged(event) {
      this.setErrors();
      const files = event.target.files;
      let errors = [];
      const fd = new FormData();
      Object.values(files).forEach((item) => {
        const error = this.isValidImage(item);
        if (error) {
          errors.push(error);
          return false;
        } else {
          fd.append("images[]", item);
        }
      });

      if (!errors.length) {
        this.params = fd;
        this.bulkImage();
      } else {
        this.setErrors({ multiple_image: errors });
      }
    },
    async bulkImage() {
      if (this.params) {
        this.setErrors();

        try {
          this.uploading = true;

          await this.setRequest({
            params: this.params,
            api: "imgUpload",
          });

          this.params = null;
          await this.fetchingData();
          this.uploading = false;
        } catch (e) {
          return this.$nuxt.error(e);
        }
      }
    },
    async deleteImage(index) {
      if (confirm(this.$t("admin.dltMsg"))) {
        try {
          this.loading = true;

          await this.deleteData({
            params: this.thumbToMain(this.thumbs[index]),
            api: "imgDelete",
          });

          await this.fetchingData();

          this.loading = false;
        } catch (e) {
          return this.$nuxt.error(e);
        }
      }
    },
    async deleteMultipleImages() {
      if (confirm("Do you really want to delete selected images?")) {
        try {
          this.loading = true;
          const imagesToDelete = this.selectedImageList.map((index) =>
            this.thumbToMain(this.thumbs[index])
          );
          await Promise.all(
            imagesToDelete.map((params) =>
              this.deleteData({
                params,
                api: "imgDelete",
              })
            )
          );

          await this.fetchingData();

          this.loading = false;
        } catch (e) {
          this.loading = false;
          return this.$nuxt.error(e);
        }
      }
    },
    setImage(data) {
      if (this.selectedImage !== data) {
        this.selectedImage = data;
      } else {
        this.selectedImage = "";
      }
    },
    setSelectedImage(index) {
      if (this.selectedImageList.includes(index)) {
        const filter = this.selectedImageList.filter((item) => item !== index);
        this.selectedImageList = filter;
      } else {
        this.selectedImageList.push(index);
      }
    },
    copyText() {
      navigator.clipboard.writeText(this.selectedImage);
    },
    thumbToMain(image) {
      return image.replace("thumb-", "");
    },
    async fetchingData() {
      this.loading = true;
      this.imageList = await this.getRequest({
        params: {},
        api: "imgAll",
      });

      this.loading = false;
    },
    ...mapActions("common", ["deleteData", "getRequest", "setRequest"]),
    ...mapActions("ui", ["setErrors"]),
  },
  async mounted() {
    await this.fetchingData();
  },
};
</script>
