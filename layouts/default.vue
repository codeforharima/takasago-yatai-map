<template>
  <v-app light>
    <v-app-bar :clipped-left="clipped" absolute flat app>
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-content>
      <v-container fluid fill-height>
        <nuxt />
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      clipped: false,
      title: '高砂 - 屋台蔵マップ'
    }
  }
}
</script>
