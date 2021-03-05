<template>
  <v-col cols="12" xs="12" sm="12" md="6" lg="4">
    <v-card class="px-8 pt-8 card" outlined>
      <h2 class="mb-1">Connect to a server</h2>
      <div class="d-flex mb-6 align-center">
        <h3>Powered by</h3>
        <img class="ml-2" width="75" src="../assets/django.png" />
      </div>
      <p>
        This Vue frontend is connected to a Django server. Below is the response
        message we receive when we ping the server:
      </p>
      <p class="response px-1">> {{ response }}</p>
      <p>
        The server ping count is stored to the database. Click below to
        <b>reset the counter</b>:
      </p>
      <div class="d-flex justify-center my-7">
        <v-btn color="grey darken-2" small @click="resetCounter">
          RESET COUNTER
        </v-btn>
      </div>
      <p>
        The <b>view</b> that resets the ping counter is located in
        <code>
          <a :href="url" target="_blank" rel="noopener">
            backend/src/counters/views.py</a
          ></code
        >:
      </p>
      <div class="editor-container">
        <prism-editor
          class="my-editor"
          v-model="codeSnippet1"
          :highlight="highlighter"
          :readonly="true"
        >
        </prism-editor>
      </div>
    </v-card>
  </v-col>
</template>

<script>
import { PrismEditor } from 'vue-prism-editor';
import 'vue-prism-editor/dist/prismeditor.min.css';

import { highlight, languages } from 'prismjs/components/prism-core';
import 'prismjs/components/prism-clike';
import 'prismjs/components/prism-javascript';
import 'prismjs/themes/prism-tomorrow.css';

const codeSnippet1 = `10 |  def reset(request):
11 |      counter = Counter.objects.last()
12 |      counter.reset()
13 |
14 |      return JsonResponse({'response': counter.value})`;

export default {
  name: 'ConnectCard',
  components: {
    PrismEditor
  },
  data: () => ({
    response: '',
    codeSnippet1: codeSnippet1,
    url: `${process.env.VUE_APP_STARTER_REPO_URL}backend/src/counters/views.py#L10-L14`
  }),
  mounted() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      fetch('/api/v1/')
        .then(res => res.json())
        .then(
          result => {
            this.response = result.response;
          },
          error => {
            console.log(error);
          }
        );
    },
    async resetCounter() {
      fetch('/api/v1/reset/')
        .then(res => res.json())
        .then(() => this.fetchData());
    },
    highlighter(code) {
      return highlight(code, languages.js);
    }
  }
};
</script>

<style scoped>
.response {
  font-family: monospace;
  color: limeGreen;
  font-size: 0.9em;
  background: black;
}
</style>
