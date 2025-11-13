<template>
  <v-app>
    <router-view></router-view>
    <br><br>
    <small class="float-right">*本程式非中國信託官方工具，純屬方便個人編輯使用。</small>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',

  components: {
    HelloWorld,
  },

  data: () => ({
    //
  }),
};
</script>
