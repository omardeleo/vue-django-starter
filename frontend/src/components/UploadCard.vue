<template>
  <v-col cols="12" xs="12" sm="12" md="6" lg="4">
    <v-card class="px-8 pt-8 card" outlined>
      <h2 class="mb-1">Store files in S3, locally</h2>
      <div class="d-flex mb-6 align-center">
        <h3>Powered by</h3>
        <img class="ml-2" width="75" src="../assets/localStackLogo.png" />
      </div>
      <p>Click below to select and upload an image from your computer.</p>
      <p>
        The image will be <b>stored in a local S3 bucket</b>, powered by
        <b><a :href="url" target="_blank" rel="noopener">LocalStack</a></b>
        - a fully functional local AWS cloud stack, and displayed below.
      </p>
      <form>
        <div>
          <input
            v-if="token"
            type="hidden"
            name="csrfmiddlewaretoken"
            value="{token}"
          />
          <div class="d-flex justify-center my-7">
            <v-btn color="grey darken-2" small @click="clickHandler">
              UPLOAD IMAGE
            </v-btn>
          </div>
          <input
            ref="uploader"
            class="d-none"
            type="file"
            accept="image/*"
            @change="submitFile"
          />
        </div>
      </form>
      <v-row>
        <v-col
          v-for="filename in filenames"
          :key="filename"
          class="d-flex child-flex"
          cols="4"
        >
          <v-img aspect-ratio="1" :src="filename"></v-img>
        </v-col>
      </v-row>
    </v-card>
  </v-col>
</template>

<script>
export default {
  name: 'UploadCard',
  data: () => ({
    filenames: [],
    url: 'https://github.com/localstack/localstack',
    token: null
  }),
  mounted() {
    this.fetchBucketFiles();
    this.getCookie();
  },
  methods: {
    clickHandler() {
      this.$refs.uploader.click();
    },
    async getCookie() {
      const response = await fetch('/api/v1/files/csrf/');
      const token = await response.json();
      this.token = token.csrfToken;
    },
    async fetchBucketFiles() {
      fetch('api/v1/files')
        .then(res => res.json())
        .then(
          result => {
            const bucket = result['Name'];
            const filenames = result['Contents']
              ? result['Contents']
                  .map(file => `${bucket}/${file['Key']}`)
                  .reverse()
              : [];
            this.filenames = filenames;
          },
          error => {
            console.log(error);
          }
        );
    },
    async submitFile(e) {
      e.preventDefault();
      const file = e.target.files;
      try {
        if (!file) {
          throw new Error('Please select an image');
        }
        const formData = new FormData();
        formData.append('file', file[0]);
        const response = await fetch('/api/v1/files/upload/', {
          method: 'POST',
          body: formData,
          headers: {
            'X-CSRFToken': this.token
          }
        });
        await response.json();
        this.fetchBucketFiles();
      } catch (error) {
        alert(error);
      }
    }
  }
};
</script>
