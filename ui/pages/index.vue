<template>
  <div class="app-wrapper">
    <el-container direction="vertical" align="center" style="align-items: center;">
      <nav-bar style="width: 100%" />
      <div class="search-bar" align="center" >
            <el-form :model="input" class="form-inline">
              <el-form-item style="width: 150px;" v-on:submit.prevent="onSubmit">
                <el-select
                  class="select"
                  v-model="input.type"
                >
                  <el-option :label="$t('All')" selected value="all"></el-option>
                  <el-option :label="$t('block')" value="block"></el-option>
                  <el-option :label="$t('tx-hash')" value="tx-hash"></el-option>
                  <el-option :label="$t('extrinsic')" value="extrinsic"></el-option>
                  <el-option :label="$t('runtime')" value="runtime"></el-option>
                </el-select>
              </el-form-item>
              <el-form-item style="width: inherit;">
                <el-input v-model="input.content" placeholder="Search by Block / Extrinsic / Tx Hash" @keyup.enter.native="onSubmit"></el-input>
              </el-form-item>
              <el-form-item style="width: 100px">
                <el-button type="primary" @click="onSubmit">{{$t('search')}}</el-button>
              </el-form-item>
            </el-form>
        </div>
      <el-container direction="horizontal" class="main" >
        <div class="main-content">
          <el-main class="output">
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
        type: "all",
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

      if (this.input.type == 'all') {
        const hashreg = /^0x[a-f0-9]+$/;
        const isHash = hashreg.test(this.input.content);
        if (isHash) {
          result = await this.$axios.$post("/api/scan/check_hash", {hash: this.input.content});
          if (!result || !result.data || !result.data.hash_type ) {
            this.output = {result: result}
            return
          }
          if (result.data.hash_type === "block" ) {
            this.output =  await this.$axios.$post("/api/scan/block", {block_hash: this.input.content});
            return
          }

          if (result.data.hash_type === "extrinsic" ) {
            this.output =  await this.$axios.$post("/api/scan/extrinsic", {hash: this.input.content});
            return
          }

          this.output = await this.$axios.$post("/api/scan/extrinsic", {extrinsic_index: result.data.extrinsic_index});            
          return
        }

        const numreg = /^[0-9]+$/;
        const isNum = numreg.test(this.input.content);
        if (isNum) {
          this.output =  await this.$axios.$post("/api/scan/block", {block_num: parseInt(this.input.content)});
          return
        }

        const indexreg = /^[0-9]+-[0-9]+$/;
        const isIndex = indexreg.test(this.input.content);
        if (isIndex) {
          this.output =  await this.$axios.$post("/api/scan/extrinsic", {extrinsic_index: this.input.content});
          return
        }

        this.output = {result: "Invalid data input"}
        return
      }

      if (this.input.type === "tx-hash") {
        const reg = /^[0-9]+$/;
        const isNum = reg.test(this.input.content);
        if (isNum) {
          this.output = {result: "Invalid data input"}
          return
        } else {
          payload = {
            hash: this.input.content,
          };
        }
        result = await this.$axios.$post("/api/scan/check_hash", payload);
        if (result && result.data && result.data.extrinsic_index ) {
          result = await this.$axios.$post("/api/scan/extrinsic", {extrinsic_index: result.data.extrinsic_index});
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
  .search-bar {
    width: 100%;
    margin: 20px;
    max-width: 800px;
  }
  .form-inline {
    display: flex;
    width: 100%;
  }
  .main-content {
    margin: 0 auto;
    width: 100%;
    max-width: 800px;
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
    text-align: left;
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
