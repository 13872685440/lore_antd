<template>
  <div class="logo">
    <router-link :to="{ name: 'index' }">
      <!-- <LogoSvg alt="logo" />-->
      <img src="/logo.png" style="width: 40px; height: 40px" alt="logo" />
      <h1 v-if="showTitle">{{ title }}</h1>
    </router-link>
  </div>
</template>

<script>
//import LogoSvg from "@/assets/logo.svg?inline";
import Vue from "vue";
import { SYSTEM_CONFIG } from "@/store/mutation-types";

export default {
  name: "Logo",
  //components: {
  //LogoSvg,
  //},
  props: {
    showTitle: {
      type: Boolean,
      default: true,
      required: false,
    },
  },
  data() {
    return {
      title: Vue.ls.get(SYSTEM_CONFIG).system_name,
    };
  },
};
</script>
