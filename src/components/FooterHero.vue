<template>
  <div class="hero-foot">
    <footer class="footer">
  <strong>candidato:</strong> Marcus Cunha (FrontEnd/UX) /
  <strong>E-mail:</strong> mystrader@gmail.com |||
  <a href="https://www.behance.net/mystrader">Portifólio </a> &
  <a href="mystrader.com">Currículo </a>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'footerHero',
  props: {},
  data () {
    return {
      isOpenedInIFrame: false
    }
  },
  created () {
    if (window.self !== window.top) {
      this.isOpenedInIFrame = true
    }
  }
}
</script>

<style scoped lang="scss">
$small: 590px;
$medium: 768px;
$large: 1024px;

.footer {
  background-color: transparent;
  padding: 3rem 1.5rem 0rem;
  position: relative;

  @media screen and (max-width: $small) {
    padding-top: 60px;
  }

  .mobile-footer {
    display: none;
    font-size: 15px;
    font-weight: 400;
    position:absolute;
    bottom: 15px;
    left: 0;
    right: 0;
    margin: 0 auto;

    a {
      color: #fd6721;
    }

    @media screen and (max-width: $small) {
      display: block;
    }

    .wallet-tag {
      font-size: 9px;
      font-weight: 600;

      span {
        color: #00d1b2;
      }
    }
  }

  .desktop-footer {
    display: block;

    .button {
      float: right;
      padding: 10px;
      margin-bottom: 10px;
    }

    @media screen and (max-width: $medium) {
      display: none;
    }
  }
}

.hide {
  display: none !important;
}
</style>
