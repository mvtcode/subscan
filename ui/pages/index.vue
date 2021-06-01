<template>
  <div class="app-wrapper">
    <el-container direction="vertical">
      <nav-bar />
      <el-container direction="horizontal" class="main">
        <div class="main-content">
          <el-main class="input">
            <div class="title">{{$t('search_title')}}</div>
            <el-form :model="input" class="demo-form-inline" label-position="top">
              <el-form-item :label="$t('search_type')">
                <el-select
                  class="select"
                  v-model="input.type"
                  :placeholder="$t('search_placeholder')"
                >
                  <el-option :label="$t('block')" value="block"></el-option>
                  <el-option :label="$t('tx-hash')" value="tx-hash"></el-option>
                  <el-option :label="$t('extrinsic')" value="extrinsic"></el-option>
                  <el-option :label="$t('runtime')" value="runtime"></el-option>
                </el-select>
              </el-form-item>
              <el-form-item :label="$t('search_content')">
                <el-input v-model="input.content"></el-input>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="onSubmit">{{$t('search')}}</el-button>
              </el-form-item>
            </el-form>
          </el-main>
          <el-main class="output">
            <div class="title">{{$t('output')}}</div>
            <client-only>
              <json-pretty class="output-json" v-if="output" :data="output"></json-pretty>
            </client-only>
          </el-main>
        </div>
      </el-container>
      <el-container direction="horizontal" class="main" v-show="input && input.type && input.type === 'tx-hash'">
        <div class="main-content">
        <el-main class="output">
            <div class="title">Block Detail</div>
            <client-only>
              <json-pretty class="output-json" v-if="block_output" :data="block_output"></json-pretty>
            </client-only>
          </el-main>
           <el-main class="output">
            <div class="title">Extrinsic Detail</div>
            <client-only>
              <json-pretty class="output-json" v-if="extrinsic_output" :data="extrinsic_output"></json-pretty>
            </client-only>
          </el-main>
        </div>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import NavBar from "~/components/NavBar.vue";
export default {
  components: {
    NavBar,
    JsonPretty: () =>
      process.client
        ? import("~/components/JsonPretty")
        : Promise.resolve({ render: (h) => h("div") }),
  },
  data() {
    return {
      input: {
        type: "",
        content: "",
      },
      output: "",
      block_output: "",
      extrinsic_output: "",
    };
  },
  methods: {
    async onSubmit() {
      this.block_output = ""
      this.extrinsic_output = ""
      let payload = {};
      let result;
      if (!this.input.content) {
        return;
      }
      if (this.input.type === "tx-hash") {
        const reg = /^[0-9]+$/;
        const isNum = reg.test(this.input.content);
        if (isNum) {
          alert('invalid hash format')
        } else {
          payload = {
            hash: this.input.content,
          };
        }
        result = await this.$axios.$post("/api/scan/check_hash", payload);
        if (result.data && result.data.data && result.data.data.block_num) {
          this.block_output = await this.$axios.$post("/api/scan/block", {block_num: result.data.data.block_num});
        }
        if (result.data && result.data.data && result.data.data.block_num && result.data.data.extrinsic_idx ) {
          this.extrinsic_output = await this.$axios.$post("/api/scan/extrinsic", {extrinsic_index: `${result.data.data.block_num}-${result.data.data.extrinsic_idx}`});
        }
      } else if (this.input.type === "block") {
        const reg = /^[0-9]+$/;
        const isNum = reg.test(this.input.content);
        if (isNum) {
          payload = {
            block_num: +this.input.content,
          };
        } else {
          payload = {
            block_hash: this.input.content,
          };
        }
        result = await this.$axios.$post("/api/scan/block", payload);
      } else if (this.input.type === "extrinsic") {
        const reg = /^[0-9]+-[0-9]+$/;
        const isNum = reg.test(this.input.content);
        if (isNum) {
          payload = {
            extrinsic_index: this.input.content,
          };
        } else {
          payload = {
            hash: this.input.content,
          };
        }
        result = await this.$axios.$post("/api/scan/extrinsic", payload);
      } else if (this.input.type === "runtime") {
        payload = {
          spec: +this.input.content,
        };
        result = await this.$axios.$post("/api/scan/runtime/metadata", payload);
      }
      this.output = result;
    },
  },
  // async fetch({ store, params, $axios }) {
  //   return store.dispatch("SetMetaData");
  // },
};
</script>
<style lang='scss' scoped>
@import "~assets/style/index.scss";
.app-wrapper {
  display: flex;
  .main {
    padding: 0 20px;
  }
  .main-content {
    margin: 0 auto;
    width: 1180px;
    display: flex;
  }
  .select {
    width: 100%;
  }
  .title {
    margin-bottom: 50px;
  }
  .input,
  .output {
    width: 50%;
    padding: 20px 20px 0 0;
  }
  .output-json {
    height: 500px;
    overflow: scroll;
    background-color: #f3f5f9;
    padding: 10px;
  }
  @media screen and (max-width: $screen-xs) {
    .main-content {
      flex-direction: column;
    }
    .input,
    .output {
      width: 100%;
      padding: 20px 20px 0 0;
    }
  }
}
</style>
<style>
html,
body,
.app-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
.app-wrapper {
  position: absolute;
  display: flex;
}
</style>
