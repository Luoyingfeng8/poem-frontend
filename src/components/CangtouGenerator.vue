<template>
  <v-container>
    <v-layout row wrap>
      <v-flex xs12>
        <v-text-field
            v-model="keyword"
            :error-messages="keywordErrors"
            :label="keywordPlaceHolder"
            required
            @input="$v.keyword.$touch()"
            @blur="$v.keyword.$touch()"
        ></v-text-field>
      </v-flex>
      <v-flex xs12>
        <v-text-field
            v-model="generateResult"
            :label="generateResultPlaceHolder"
            readonly
        ></v-text-field>
      </v-flex>

      <v-flex xs6 lg2>
        <v-switch
            v-model="isQiyan"
            :label="isQiyanSwitch"
        ></v-switch>
      </v-flex>

      <v-flex xs12>
        <v-btn @click="generateCangtou" color="success">生成藏头诗</v-btn>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script lang="ts">
  import Vue from "vue";
  import {validationMixin} from "vuelidate";
  import {required, maxLength, minLength, sameAs} from "vuelidate/lib/validators";
  import config from "@/config.ts";
  import {get, post} from "@/helpers.ts";

  export default Vue.extend({
    name: "CangtouGenerator",
    mixins: [validationMixin],
    validations: {
      keyword: {required, maxLength: maxLength(4)},
    },
    data: () => ({
      // Single
      keyword: "",
      generateResult: "",
      isQiyan: false,
    }),
    computed: {
      isQiyanSwitch() {
        if (this.isQiyan) {
          return "七言";
        } else {
          return "五言";
        }
      },
      keywordErrors() {
        const errors: any[] = [];
        if (!this.$v.keyword!.$dirty) {
          return errors;
        }
        !this.$v.keyword!.maxLength && errors.push("关键词长度不能超过 4");
        !this.$v.keyword!.required && errors.push("请输入关键词");
        return errors;
      },
      keywordPlaceHolder() {
        return "请输入关键词";
      },
      generateResultPlaceHolder() {
        return "生成结果";
      },
      generateType() {
        if (this.isQiyan) {
          return "7";
        } else {
          return "5";
        }
      }
    },
    methods: {
      async generateCangtou() {
        this.$v.$touch();
        if (this.$v.keyword!.$invalid) {
          return;
        } else {
          this.generateResult = "";
          try {
            const response = await post<{
              result: boolean,
              code: number,
              msg: string,
              data: string,
            }>(config.baseUrl + "acrostic",
              {
                input: this.keyword,
                type: this.generateType,
              });
            console.log(response.parsedBody);
            if (response.parsedBody!.result) {
              this.showInfo(response.parsedBody!.msg);
              // console.log(response.parsedBody.data);
              this.generateResult = response.parsedBody.data;
            } else {
              this.showError(response.parsedBody!.msg);
            }
          } catch (response) {
            this.showError("出错啦！详细信息：" + response);
          }
        }
      },
      showInfo(msg: string) {
        this.$store.dispatch("showInfo", msg);
      },
      showError(msg: string) {
        this.$store.dispatch("showError", msg);
      },
    },
  });
</script>

<style scoped lang="stylus">
  .fade-enter-active, .fade-leave-active
    transition: opacity .5s

  .fade-enter, .fade-leave-to
    opacity: 0
</style>
