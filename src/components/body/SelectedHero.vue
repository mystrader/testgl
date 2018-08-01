<template>
    <div class="columns selected-section" :class="{'no-padding': isOpenedInIFrame}">
      <router-link to="/" class="nav-item">
        <div class="return-action" :class="{'return-action-iframe': isOpenedInIFrame}">
          <icon name="chevron-left" class="arrow-left" :class="{'arrow-left-iframe': isOpenedInIFrame}" height="22" width="22"></icon>
          <p class="return-text" :class="{'hide': isOpenedInIFrame}">Retornar para votação</p>
        </div>
      </router-link>
      <div class="column is-12 image-section">
        <img :src="`/static/${selectedCryptoCurrency.id}_large_logo.png`" class="cryptoCurrency-image" :class="{'cryptoCurrency-image-iframe': isOpenedInIFrame}"/>
        <h1 class="cryptoCurrency-title" :class="{'cryptoCurrency-title-iframe': isOpenedInIFrame}">{{ selectedCryptoCurrency.website }}</h1>

        <p class="cryptoCurrency-description" :class="{'cryptoCurrency-description-iframe': isOpenedInIFrame}">{{ selectedCryptoCurrency.description }}</p>

            <a class="button is-rounded is-success is-large" :class="{'hide': isOpenedInIFrame}" @click="toggleModal">
               <i class="fas fa-check"></i>
              <span>Vote em mim</span>
            </a>
      </div>


 <div class="modal" :class="{'is-active': modalActive}">
      <div class="modal-background" @click="toggleModal"></div>
        <div class="modal-card">
          <header class="modal-card-head"></header>
          <section class="modal-card-body">
            <div class="content">
              <h2 >Sou <strong>{{ selectedCryptoCurrency.website }}</strong> e estou muito feliz pelo seu voto</h2>
            </div>
            <router-link to="/" >
            <button class="button is-danger is-rounded" >Seguir na folia</button>
            <!-- @click="toggleModal" -->
            </router-link>
          </section>
          
        </div>
    </div>



    </div>




    
</template>

<script>
import { store } from '../../store.js'
const fiatCurrencies = [ 'Colombina', 'Arlequim', 'Pierrô', 'Capitano', 'Briguela', 'EUR', 'GBP', 'HKD', 'IDR', 'INR', 'JPY', 'USD', 'KRW', 'MXN', 'RUB' ]
const cryptoCurrencyData = require('../../cryptocurrency-data.json')

export default {
  name: 'selectedHero',
  data () {
    return {
      modalActive: false,
      sharedState: store.state,
      selectedCryptoCurrency: {},
      fiatCurrencies: fiatCurrencies,
      selectedFiatCurrency: fiatCurrencies[11],
      positivePercentChange: true,
      negativePercentChange: false,
      dropDownOpen: false,
      isOpenedInIFrame: false
    }
  },
  created () {
    this.selectCryptoCurrency()

    if (window.self !== window.top) {
      this.isOpenedInIFrame = true
    }
  },
  watch: {
    $route () {
      this.selectCryptoCurrency()
    }
  },
  computed: {
    globalMarketCapUSD () {
      return this.sharedState.totalMarketCapUSD
    },
    selectedCryptoCurrencyMarketCap () {
      return Number(this.selectedCryptoCurrency.market_cap_usd)
    },
    percentageOfMarketCap () {
      const percentageOfMarketCap = Math.round((this.selectedCryptoCurrency.market_cap_usd / this.sharedState.totalMarketCapUSD) * 100)
      return `~ ${percentageOfMarketCap}% of the global cryptocurrency market cap!`
    }
  },
  methods: {
    toggleModal () {
      this.modalActive = !this.modalActive
    },
    toggleDropDown () {
      this.dropDownOpen = !this.dropDownOpen
    },
    selectCryptoCurrency () {
      let cryptoCurrency
      if (this.sharedState.cryptoCurrencies.length === 0 || !this.sharedState.cryptoCurrencies) {
        this.axios.get(`https://api.coinmarketcap.com/v1/ticker/${this.$route.params.id}/`)
          .then(response => {
            cryptoCurrency = response.data[0]
            cryptoCurrency = this.manipulateCryptoCurrency(cryptoCurrency)
            this.selectedCryptoCurrency = this.addImageAndDescription(cryptoCurrency)
          })
      } else {
        cryptoCurrency = this.sharedState.cryptoCurrencies.filter((obj) => {
          return obj.id === this.$route.params.id
        })[0]
        this.selectedCryptoCurrency = this.manipulateCryptoCurrency(cryptoCurrency)
      }
    },
    selectFiatCurrency (fiatCurrencyEvent) {
      this.selectedFiatCurrency = fiatCurrencyEvent.target ? fiatCurrencyEvent.target.value : fiatCurrencyEvent
      this.axios.get(`https://api.coinmarketcap.com/v1/ticker/${this.selectedCryptoCurrency.id}/?convert=${this.selectedFiatCurrency}`)
        .then(cryptoCurrency => {
          this.toggleDropDown()
          this.selectedCryptoCurrency.selectedPrice = Number(cryptoCurrency.data[0]['price_' + this.selectedFiatCurrency.toLowerCase()]).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })
          this.selectedCryptoCurrency.selectedMarketCap = Number(cryptoCurrency.data[0]['market_cap_' + this.selectedFiatCurrency.toLowerCase()]).toLocaleString()
        })
    },
    manipulateCryptoCurrency (cryptoCurrency) {
      cryptoCurrency.selectedPrice = Number(cryptoCurrency.price_usd).toFixed(2)
      cryptoCurrency.selectedSupply = Number(cryptoCurrency.available_supply).toLocaleString()
      cryptoCurrency.selectedMarketCap = Number(cryptoCurrency.market_cap_usd).toLocaleString()
      return cryptoCurrency
    },
    addImageAndDescription (cryptoCurrency) {
      cryptoCurrency.id = cryptoCurrency.id in cryptoCurrencyData ? cryptoCurrency.id : undefined
      cryptoCurrency.image = `${cryptoCurrency.id}_image`
      cryptoCurrency.nameperson = cryptoCurrencyData[cryptoCurrency.id].nameperson
      cryptoCurrency.description = cryptoCurrencyData[cryptoCurrency.id].description
      cryptoCurrency.website = cryptoCurrencyData[cryptoCurrency.id].website
      cryptoCurrency.paper = cryptoCurrencyData[cryptoCurrency.id].paper
      cryptoCurrency.github = cryptoCurrencyData[cryptoCurrency.id].github

      cryptoCurrency.positivePercentChange = !(cryptoCurrency.percent_change_24h.indexOf('-') > -1)
      cryptoCurrency.percentChange24h = cryptoCurrency.percent_change_24h.replace(/^-/, '')
      return cryptoCurrency
    }
  }
}
</script>

<style scoped lang="scss">
$small: 590px;
$medium: 768px;
$large: 1024px;

.selected-section {
  position: relative;
  align-items: center;

  @media screen and (max-width: $medium) {
    padding-top: 30px;
  }

  .return-action {
    position: absolute;
    left: 30px;
    color: #FFF;
    cursor: pointer;
    z-index: 10;

    .arrow-left {
      padding-right: 5px;
    }

    .arrow-left-iframe {
      color: #fd6721;
    }

    .return-text {
      font-size: 11px;

      @media screen and (max-width: $medium) {
        display: none;
      }
    }
  }

  .return-action-iframe {
    top: -10px;
  }

  .image-section {
    position: relative;

    .cryptoCurrency-image {
      height: 200px;
      }

    .cryptoCurrency-image-iframe {
      height: 80px;
    }

    .cryptoCurrency-title {
      font-size: 20px;
      font-weight: 600;
    }

    .cryptoCurrency-title-iframe {
      font-size: 16px;
    }

    .tag-iframe {
      font-size: 0.63rem;
    }

    .cryptoCurrency-description {
      font-size: 14px;
      line-height: 1.65;
      max-width: 450px;
      display: block;
      margin: 0 auto;
      padding: 20px 0 30px;

      @media screen and (max-width: $medium) {
        font-size: 12px;
        max-width: 300px;
      }
    }

    .cryptoCurrency-description-iframe {
      font-size: 11px;
      padding-top: 10px;
      padding-bottom: 15px;
      line-height: 1.5;
      max-width: 425px;
    }

    .icons-section {
      height: 0px;

      .icon {
        margin: 0px 30px;
        -o-transition: .5s;
        -ms-transition: .5s;
        -moz-transition: .5s;
        -webkit-transition: .5s;
        transition: .5s;

        &:hover {
          color: #fd6721;
        }
      }
    }
  }

  .information-section {
    position: relative;
    padding-top: 25px;

    .dropdown-section {
      position: absolute;
      right: 50px;

      @media screen and (max-width: $medium) {
        right: 20px;
      }

      .control {
        text-align: center;

        .button {
          font-size: 11px;
          width: 50px;
          color: #1e1335;
          position: relative;

          &:focus {
            border-color: #FFF;
          }

          .arrow-icon {
            position: absolute;
            right: 2px;
            bottom: 1px;
            color: #fd6721;
          }
        }

        .box {
          padding: 7px;
          background-color: transparent;
        }

        .dropdown {
          width: 50px;
          box-shadow: 0 0 3px 0 #fd6721;
          display: none;
          left: 0;
          position: absolute;
          top: 100%;
          z-index: 1000;
          height: 150px;
          overflow-y: scroll;
          background-color: #FFF;

          ul {
          }

          &.is-open {
            display: block;
          }

          .nav-item {
            font-size: 11px;
            color: #1e1335;
            -o-transition: .5s;
            -ms-transition: .5s;
            -moz-transition: .5s;
            -webkit-transition: .5s;
            transition: .5s;

            &:hover {
              color: #fd6721;
            }
          }
        }
      }
    }

    .price-section {
      text-align: left;
      padding: 20px 0;

      @media screen and (max-width: $medium) {
        text-align: center;
      }

      .price-tag {
      }

      .price-select-tag {
        display: inline-block;
      }

      .control {
        display: inline-block;
        float: right;
        margin-right: 30px;
        font-size: 0.80rem;
        .select {
          select {
            padding-right: 1.2em;
          }
          &:after {
            right: 0.55em;
            border: 1px solid #fd6721;
            border-top: 0;
            border-right: 0;
          }
        }

        @media screen and (max-width: $medium) {
          display: block;
          text-align: center;
          float: none;
          margin-right: 0px;
        }
      }

      .price-amount {
        font-size: 40px;

        .positive-percent-change {
          color: #00d1b2;
        }

        .negative-percent-change {
          color: #ff3860;
        }

        .percent-tooltip {
          color: #FFF;
          text-align: center;
          font-size: 10px;
          font-weight: 500;
          cursor: initial;
        }

        @media screen and (max-width: $medium) {
          font-size: 30px;
        }
      }

      .market-cap-price-amount {
        display: inline-block;
      }

      .doughnut-chart {
        display: block;
        text-align: center;
        font-weight: 500;
        width: 50px;
        margin: 0 auto;
      }

      .price-amount-iframe {
        font-size: 20px;
        font-weight: 600;
      }
    }

    .price-select-section {
      @media screen and (min-width: $medium) {
      }
    }

    .price-section-iframe {
      padding: 10px 0;
    }
  }

  .information-section-iframe {
  }
}

.tooltip {
  position: relative;
  display: inline-block;
  border-bottom: 1px dotted black;

  &:hover .tooltiptext {
    visibility: visible;
  }

  .tooltiptext {
    visibility: hidden;
    width: 120px;
    background-color: #383838;
    font-size: 11px;
    font-weight: 500;
    color: #fff;
    text-align: center;
    border-radius: 6px;
    padding: 5px 0;

    /* Position the tooltip */
    position: absolute;
    z-index: 1;
    top: 100%;
    left: 50%;
    margin-left: -60px;
  }
}

.doughnut-tooltip {
  .tooltiptext {
    top: 110%;
  }
}

.hide {
  display: none !important;
}

.show {
  display: block !important;
}

.no-padding {
  padding: 0px;
}

.transparent {
  background-color: transparent !important;
}

.font-white {
  color: #FFF !important;
}

@-webkit-keyframes spinner {
    from
    {
        -webkit-transform: rotateY(0deg);
    }
    to {
        -webkit-transform: rotateY(-360deg);
    }
}
@keyframes spinner {
    from {
        -moz-transform: rotateY(0deg);
        -ms-transform: rotateY(0deg);
        transform: rotateY(0deg);
    }
    to
    {
        -moz-transform: rotateY(-360deg);
        -ms-transform: rotateY(-360deg);
        transform: rotateY(-360deg);

    }
}

::-webkit-scrollbar {
    -webkit-appearance: none;
    width: 7px;
}
::-webkit-scrollbar-thumb {
    border-radius: 4px;
    background-color: #FFF;
    -webkit-box-shadow: 0 0 1px rgba(255,255,255,.5);
}


.modal {
  .modal-card {
    border-radius: 10px;
    max-width: 500px;

     @media screen and (max-width: $medium) {
      margin: 0px;
    }

    .modal-card-head {
      display: block;
      background-color: #FFF;
      border-bottom: 0px;
      padding-bottom: 0px;

      .modal-logo {
        height: 80px;
        display: block;
        margin: 0 auto;
        margin-bottom: 10px;

        @media screen and (max-width: $medium) {
          height: 40px;
        }
      }

      .modal-card-title {
        font-size: 25px;
        letter-spacing: 5px;
      }
    }

    .modal-card-body {
      font-family: Nunito, sans-serif;
      color: #000;
      padding: 0px 20px;
      padding-bottom: 20px;

      p {
        font-size: 14px;

        @media screen and (max-width: $medium) {
          font-size: 12px;
        }

        a {
          color: #fd6721;
          font-weight: 700;
          text-decoration: none;
        }
      }

      h3 {
        color: #fd6721;
        font-size: 20px;
        font-weight: 600;
        margin-bottom: 10px;
      }

      #app-logo {
        height: 60px;
      }

      #vue-logo {
        height: 60px;

        -webkit-animation-name: spinner;
        -webkit-animation-timing-function: linear;
        -webkit-animation-iteration-count: infinite;
        -webkit-animation-duration: 30s;
        animation-name: spinner;
        animation-timing-function: linear;
        animation-iteration-count: infinite;
        animation-duration: 30s;
        -webkit-transform-style: preserve-3d;
        -moz-transform-style: preserve-3d;
        -ms-transform-style: preserve-3d;
        transform-style: preserve-3d;

        @media screen and (max-width: $medium) {
          height: 40px;
        }
      }

      .is-success {
        background-color: #fd6721;

        @media screen and (max-width: $medium) {
          display: none;
        }
      }
    }

    .modal-card-foot {
      font-family: Nunito, sans-serif;
      padding-bottom: 1px;
      background-color: transparent;

      @media screen and (max-width: $medium) {
        padding-top: 5px;
      }

      .content {
        color: #FFF;
        display: block;
        margin: 0 auto;

        @media screen and (max-width: $medium) {
          display: none;
        }

        .footer-title {
          font-size: 15px;
          font-weight: 400;
        }

        .footer-social-media {
          height: 30px;
          color: #fd6721;

          .icon {
            text-decoration: none;
            -webkit-transform: scale(0.8);
            -moz-transform: scale(0.8);
            -o-transfrom: scale(0.8);
            -webkit-transition-duration: 0.5s;
            -moz-transition-duration: 0.5s;
            -o-transition-duration: 0.5s;
            transition: 0.5s;

            &:hover {
              color: #4fc08d;
            }
          }
        }

        .wallet-content {
          display: block;
          margin: 0 auto;
          font-size: 11px;

          .wallet-tag {
            font-weight: 600;
            cursor: pointer;
            margin-bottom: 0px;
            -o-transition: .5s;
            -ms-transition: .5s;
            -moz-transition: .5s;
            -webkit-transition: .5s;
            transition: .5s;

            &:hover {
              color: #fd6721;
            }
          }

          .wallet-id {
            color: #00d1b2;
          }
        }
      }
    }
  }

  .modal-close {
    background-color: #fd6721;

    @media screen and (max-width: $medium) {
      position: absolute;
      top: 15px;
      right: 5px;
    }
  }



}


</style>
