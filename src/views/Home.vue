<!-- eslint-disable vue/no-use-v-if-with-v-for -->
<template>
  <div class="home">
    <img
      alt="map"
      src="../assets/world.svg"
      class="map"
      ref="map"
      @load="onMapLoad"
      style="height:100vh"
    >
    <div v-if="requestEnded">
      <div
        class="map-point"
        v-for="(trans, key) in transactions"
        :style="trans.mapPosition"
        :key="'trans.city' + key"
      >
        <div class="map-point-details">
          <img :src="require('../assets/icons/' + trans.type + '.svg')" class="icon">
          {{ trans.value + trans.currencySymbol }}
        </div>
        •
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable prefer-template */
import axios from 'axios';
import currencies from '../assets/currencies.json';

export default {
  name: 'Home',
  data() {
    return {
      mapWidth: 0,
      mapHeight: 0,
      transactions: [],
      requestEnded: false,
    };
  },
  methods: {
    onMapLoad() {
      this.mapHeight = this.$refs.map.height;
      this.mapWidth = this.$refs.map.width;
      this.requestTransactions();
      const ctx = this;
      setInterval(() => {
        ctx.requestTransactions();
      }, 3000);
      setInterval(() => {
        // cleaning up the first 10 transactions everytime
        ctx.transactions = ctx.transactions.splice(0, 9);
      }, 5000);
    },
    getRandomDelay() {
      const max = 3000;
      const min = 0;
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
    addParis() {
      const coord = this.calculateCoordinates(48.8534, 2.3488);
      const mapPos = {
        'margin-left': coord[0] + 'px',
        'margin-top': coord[1] + 'px',
        animation: 'dot 2s ease-out 0ms infinite',
      };
      const finalObject = {
        currency: 'EUR',
        mapPosition: mapPos,
      };
      this.transactions.push(finalObject);
    },
    requestTransactions() {
      // using https://cors-anywhere.herokuapp.com/ to bypass Revolut cross-origin CORS
      // thanks Rob--W <3
      // first with reset transactions
      axios.get('http://localhost:8081/https://www.revolut.com/api/transactions/last')
        .then((resp) => {
          const tempTransactions = resp.data;
          tempTransactions.forEach((trans) => {
            // console.log(trans.city);
            const coord = this.calculateCoordinates(trans.location.lat, trans.location.lon);
            const randomDelay = this.getRandomDelay();
            // console.log(randomDelay);
            const mapPos = {
              'margin-left': coord[0] + 'px',
              'margin-top': coord[1] + 'px',
              animation: 'dot 2s ease-out ' + randomDelay + 'ms forwards',
            };
            const finalObject = {
              currency: trans.amount.ccy,
              mapPosition: mapPos,
              type: trans.type,
              currencySymbol: currencies[trans.amount.ccy].symbol_native,
              value: trans.amount.value,
            };
            this.transactions.push(finalObject);
          });
          this.requestEnded = true;
        })
        .catch((err) => {
          console.warn(err);
        });
    },
    calculateCoordinates(lat, long) {
      // very useful resource
      // https://stackoverflow.com/questions/14329691/convert-latitude-longitude-point-to-a-pixels-x-y-on-mercator-projection
      let x = (long + 180) * (this.mapWidth / 360);

      // convert from degrees to radians
      const latRad = lat * (Math.PI / 180);
      // get y value
      const mercN = Math.log(Math.tan((Math.PI / 4) + (latRad / 2)));
      let y = (this.mapHeight / 2) - (this.mapWidth * (mercN / (2 * Math.PI)));

      x = Math.round(x);
      y = Math.round(y);

      return [x, y];
    },
  },
};
</script>

<style lang="scss">
.map {
  position: absolute;
  left: 0;
  &-point {
    position: absolute;
    color: rgb(0, 117, 235);
    opacity: 0;
    // calculated from paris this.calculateCoordinates(48.8534, 2.3488);
    // on a macbook 13 2017 retina display
    top: 18.45%;
    left: -2.45%;
    &-details {
      position: absolute;
      left: -6px;
      top: -15px;
      width: 100px;
      .icon {
        width: 20px;
        float: left;
        margin-right: 7px;
      }
    }
  }
}
@keyframes dot {
  from { opacity: 1; }
  90% { opacity: 1; }
  to { opacity: 0; }
}
</style>
