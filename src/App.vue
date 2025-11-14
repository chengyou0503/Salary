<template>
  <v-app>
    <v-app-bar app dark color="primary">
      <v-toolbar-title>轉帳系統</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text to="/">批次轉帳</v-btn>
      <v-btn text to="/salary-transfer">薪資轉帳</v-btn>
    </v-app-bar>
    <v-main>
      <router-view></router-view>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    //
  }),
};
</script>
