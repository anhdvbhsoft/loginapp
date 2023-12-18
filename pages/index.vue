<template>
  <Login></Login>
</template>

<script lang="ts">
import Vue from "vue";
import Login from "~/components/Login.vue";
export default Vue.extend({
  name: "IndexPage",
  components: { Login },
});
</script>

<style lang="css">
body {
  background-image: url("https://mega.com.vn/media/news/0306_Hinh-nen-den-4k78.jpg");
}
</style>
