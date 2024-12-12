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
        <!-- <ajax-button name="download" class="primary-btn" :text="$t("category.download")" /> -->
        <inline-pop-over
          :arrow="true"
          :left="true"
          class="bulk-action"
          ref="bulkAction"
        >
          <template v-slot:button>
            {{ $t("title.act") }}
          </template>
          <template v-slot:content style="left: 0 !important">
            <button
              :disabled="selectedImageList.length < 1"
              class="outline-btn"
            >
              {{ $t("category.download") }}
            </button>
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
      <div class="dply-felx gap-10 j-left f-wrap mt-md-15">
        <dropdown
          :selectedKey="sortOrder"
          :options="{
            default: { title: 'Sort By' },
            az: { title: 'A to Z' },
            za: { title: 'Z to A' },
          }"
          style="width: 110px"
          :rounded="true"
          @clicked="dropdownChange(true, $event)"
        />
        <form class="search-input media-search">
          <input type="text" :placeholder="$t('list.sh')" v-model="search" />
          <!-- <button class="primary-btn">{{ $t("list.srch") }}</button> -->
        </form>
      </div>
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
          <!-- <p
            @click.stop="setSelectedImage(index)"
            :class="selectedImageList.includes(index) ? 'check' : 'uncheck'"
          >
            {{ selectedImageList.includes(index) ? "✓" : "" }}
          </p> -->
          <button class="" @click.stop="setSelectedImage(index)">
            {{ selectedImageList.includes(index) ? "✓" : "" }}
          </button>
        </div>
      </div>
    </div>
    <!-- Image popover  -->
    <pop-over
      v-if="selectedImage"
      title="Image data"
      @close="
        $emit('close');
        selectedImage = null;
      "
      elem-id="media-pop-over"
      :layer="true"
    >
      <template v-slot:content>
        <div style="display: flex">
          <img
            :src="getImageURL(selectedImage)"
            style="max-width: 220px; margin-left: auto; margin-right: auto"
            alt=""
          />
        </div>
        <div
          style="
            margin-top: 20px;
            padding-top: 20px;
            padding-bottom: 10px;
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
      </template>
    </pop-over>
    <!-- <input
      type="file"
      accept="image/*"
      @change="imageChanged"
      ref="fileInput"
      style="display: none"
      multiple
    /> -->
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
import InlinePopOver from "~/components/InlinePopOver";
import PopOver from "~/components/PopOver";
import Dropdown from "~/components/Dropdown";

export default {
  name: "images",
  middleware: ["common-middleware", "auth"],
  data() {
    return {
      loading: false,
      imageList: [],
      originalImageList: [],
      selectedImageList: [],
      selectedImage: "",
      uploading: false,
      search: "",
      sortOrder: "az",
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
      if (!this.imageList || !Array.isArray(this.imageList)) return [];
      return this.imageList.filter((image) => {
        const isNotThumb = !image.startsWith("thumb-");
        const matchesSearch = this.thumbToMain(image)
          .toLowerCase()
          .includes(this.search.toLowerCase());
        return isNotThumb && matchesSearch;
      });
    },
  },
  methods: {
    // async imageChanged(event) {
    //   this.setErrors();
    //   const files = event.target.files;
    //   let errors = [];
    //   const fd = new FormData();
    //   Object.values(files).forEach((item) => {
    //     const error = this.isValidImage(item);
    //     if (error) {
    //       errors.push(error);
    //       return false;
    //     } else {
    //       fd.append("images[]", item);
    //     }
    //   });

    //   if (!errors.length) {
    //     this.params = fd;
    //     this.bulkImage();
    //   } else {
    //     this.setErrors({ multiple_image: errors });
    //   }
    // },
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
    dropdownChange(_, e) {
      this.sortOrder = e.key;
      if (e.key !== "default") {
        const filterd = this.thumbs.sort((a, b) => {
          if (e.key === "az") {
            return a.localeCompare(b); // A-Z sorting
          } else if (e.key === "za") {
            return b.localeCompare(a); // Z-A sorting
          }
        });
        this.imageList = filterd;
      } else {
        this.imageList = this.originalImageList;
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
      const data = await this.getRequest({
        params: {},
        api: "imgAll",
      });

      this.imageList = data.sort((a, b) => a.localeCompare(b));
      this.originalImageList = data.sort((a, b) => a.localeCompare(b));
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
