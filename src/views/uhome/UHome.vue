<template>
  <div id="uhome">
    <el-container>
      <el-header class="header">
        <headers></headers>
      </el-header>
      <el-main>
        <keep-alive >
          <router-view
            :key="
              $route.name +
                ($route.query.sid ||
                  $route.query.musicid ||
                  $route.query.vid ||
                  '')
            "
          ></router-view>
        </keep-alive>
      </el-main>
      <find></find>
      <el-footer>
        <footers></footers>
      </el-footer>
    </el-container>
  </div>
</template>

<script>
import Headers from "./header/Headers.vue";
import Footers from "./footer/Footers.vue";
import Find from "views/uhome/container/find/Find.vue";
export default {
  name: "UHome",
  components: {Find, Headers, Footers },
  methods: {}
};
</script>

<style scoped>
.header {
  background-color: #242424;
  border-bottom: 1px solid #000;
}

.el-main {
  padding: 0;
}

.el-footer {
  padding: 0;
}
</style>
