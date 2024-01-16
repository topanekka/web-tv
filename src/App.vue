<template>
  <Nav :tvs="tvs" :active="url"></Nav>
  <component :is="currentView" :value="url" :track="caption" />
</template>

<script setup>
import { listTv } from "./api";
import { parse, suffix } from "./utils/tvlistsupport";
import { ref, computed, onMounted } from "vue";
import Home from "./views/Index.vue";
import NotFound from "./views/NotFound.vue";
import Nav from "./components/Nav.vue";
const routes = {
  "/": Home,
};
const currentPath = ref(window.location.hash);
const url = ref("");
const tvs = ref([]);
const caption = ref("");
window.addEventListener("hashchange", () => {
  currentPath.value = window.location.hash;
});
const currentView = computed(() => {
  if (currentPath.value.slice(1).includes("?")) {
    url.value = new URLSearchParams(
      currentPath.value.slice(currentPath.value.indexOf("?"))
    ).get("url");
    caption.value = new URLSearchParams(
      currentPath.value.slice(currentPath.value.indexOf("?"))
    ).get("caption");
  }
  return routes[currentPath.value.slice(1).split("?")[0] || "/"] || NotFound;
});

onMounted(() => {
  async function initTvList() {
    let params = new URLSearchParams(window.location.search);
    caption.value = params.get("caption");
    let url0 = params.get("url");
    let tvlistUrl = params.get("s");
    if (!(url0 || tvlistUrl)) {
      params = new URLSearchParams(window.location.hash.replace("#/", ""));
      url0 = params.get("url");
      tvlistUrl = params.get("s");
    }
    tvlistUrl = tvlistUrl || localStorage.getItem("tvlistUrl") || "https://sonydaimenew.akamaized.net/hls/live/2022320/DAI16ME_CPM3/master_mdh.m3u8?hdnea=st=1705387778~exp=1705388078~acl=*~id=5aa64e90-5f98-422c-8974-284acbc95e3a~hmac=e23ae6f91a56f062aa46f7ab1e897564c01a29343b4a4c37406b01651948d832&originpath=/linear/hls/pb/event/wN0bVs8wS9-yv6vL9G-tGQ/stream/7c9c1229-91f0-4c4b-86ae-3f8d923beea7:TPE/master.m3u8";
    let suffixName = tvlistUrl && suffix(tvlistUrl);
    if (tvlistUrl) {
      if (suffixName == "m3u8") {
        suffixName = "m3u";
      }
      localStorage.setItem("tvlistUrl", tvlistUrl);
    }
    url.value = url0;
    // tvurl
    let d = await listTv(tvlistUrl);
    // console.log(d);
    tvs.value = parse(d.data, suffixName);
    if (!url.value) {
      url.value = tvs.value[1].url;
      caption.value = tvs.value[1].caption;
    }
  }
  initTvList();
});
</script>
