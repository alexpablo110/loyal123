<template>
  <div id="app">
    <div class="container-fluid">
      <h1>POC: XSL Editor with Live Preview</h1>
      <button @click="renderPreview">Render</button>

      <h2 class="mt-5">XML to render</h2>
      <input type="file" @change="processXml($event.target.files[0])" />

      <div class="row mt-5">
        <div class="col">
          <h2>XSL</h2>
          <prism-editor v-model="xsl" language="xml" line-numbers />
        </div>

        <div class="col">
          <h2>Preview</h2>
          <iframe
            ref="preview"
            height="100%"
            width="100%"
            frameborder="0"
          ></iframe>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import PrismEditor from "vue-prism-editor";
import { parseXml } from "./XmlParser";
import { renderXslToIframe } from "./XslRenderer";

export default {
  name: "App",
  data() {
    return {
      xsl: null,
      xml: null,
      timer: null,
    };
  },
  components: {
    PrismEditor,
  },
  methods: {
    processXml(file) {
      let reader = new FileReader();
      reader.onload = (res) => {
        this.xml = res.target.result;
        if (this.xsl) {
          this.renderPreview();
        }
      };
      reader.readAsText(file);
    },
    async renderPreview() {
      console.log("render");
      let xml = parseXml(this.xml);
      let xsl = parseXml(this.xsl);
      renderXslToIframe(xml, xsl, this.$refs.preview.contentWindow.document);
    },
    handleRenderTimeout() {
      if (this.timer) {
        clearTimeout(this.timer);
        this.timer = null;
      }
      this.timer = setTimeout(() => {
        this.renderPreview();
      }, 1200);
    },
  },
  watch: {
    xsl() {
      this.handleRenderTimeout();
    },
  },
};
</script>

<style>
.col {
  max-height: 78vh;
}
</style>
