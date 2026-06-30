<template>
  <!-- Google font -->
  <link
    href="https://fonts.googleapis.com/css?family=Montserrat:300,300i,400,400i,500,500i,600,600i,700,700i&display=swap"
    rel="stylesheet"
  />

  <!-- Page Preloder -->
  <div id="preloder">
    <div class="loader"></div>
  </div>

  <Header />
  <Popup />

  <router-view />

  <Footer />

  <ChatFab />

  <!--====== Javascripts & Jquery ======-->
</template>

<script>
import Header from "@/layout/Header.vue";
import Footer from "@/layout/Footer.vue";
import Popup from "@/layout/Popup.vue";
import ChatFab from "@/components/ChatFab.vue";

export default {
  name: "AppPage",
  components: {
    Header,
    Footer,
    Popup,
    ChatFab,
  },
  mounted() {
    const urlParams = new URLSearchParams(window.location.search);
    const lang = urlParams.get("lang");
    if (lang) {
      this.$i18n.locale = lang;
      const url = new URL(window.location.href);
      url.searchParams.delete("lang");
      window.history.replaceState({}, "", url.toString());
      localStorage.setItem("lang", lang);
    }

    $(".loader").fadeOut();
    $("#preloder").delay(400).fadeOut("slow");

    $(".set-bg").each(function () {
      var bg = $(this).data("setbg");
      $(this).css("background-image", "url(" + bg + ")");
    });
  },
};
</script>
