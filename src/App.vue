<template>
  <v-app>
    <v-app-bar
      app
      style="height: 70px;"
      color="white"
      dark
      flat
    >
      <div class="container" style="display: flex; justify-content: center;">
        <h1 style="position: relative;" class="app_title">{{page_title}}</h1>

        <div style="width: 180px;"></div>

        <!--This will refresh the data -->
        <v-btn
          style="position: relative;" 
          :ripple="false"
          id="no-background-hover"
          class="app_title_button"
          icon
          @click="reload=!reload; mlbDataFetch(); nbaDataFetch();"
        >

          <v-icon :color="topIconColor" :class="icon_class" large>fa-sync-alt</v-icon>
        </v-btn>
      </div>
    </v-app-bar>
    
        <v-main :class="background_class">
            <v-snackbar
              style=""
              v-model="reload"
              :color="topIconColor"
              :timeout="750"
              centered
              rounded="pill"
              >
              Games have been refreshed!           
            </v-snackbar>
            <transition name="fade" mode="out-in">
            <sports-layout v-if="top_level_navigation == 'sports'" key="sports"/>
            <info-layout v-if="top_level_navigation == 'info'" key="info"/>
            </transition>
        </v-main>
    <vue-footer @nav_clicked="navigation"/>
  </v-app>
</template>

<script>

import vueFooter from './components/btn_footer';

import infoLayout from './components/info/layout';
import sportsLayout from './components/sports/layout';


export default {
  name: 'App',

  components: {
    vueFooter,
    sportsLayout,
    infoLayout
  },

  data: () => ({
    page_title: "Scores",
    top_level_navigation: 'sports',
    reload: false,
  }),
  computed: {
    icon_class: function() {
      if (this.reload == true)
        return "app_title_button_icon spin_icon"
      else 
        return "app_title_button_icon"
    },
    background_class: function() {
      if (this.top_level_navigation == 'sports' && this.$store.state.sport_checkbox) {
        // Basketball
        return "basketball_background"
      }
      else if (this.top_level_navigation == 'sports' && !this.$store.state.sport_checkbox) {
        // Baseball
        return "baseball_background"
      }
      else if (this.top_level_navigation == 'info') {
        // Info
        return "info_background"
      }
      else 
        return "info_background"
    },
    topIconColor: function() {
      if (this.top_level_navigation == 'sports' && this.$store.state.sport_checkbox) {
        // Basketball
        return "rgba(255,132,0,1)"
      }
      else if (this.top_level_navigation == 'sports' && !this.$store.state.sport_checkbox) {
        // Baseball
        return "rgba(170,10,10,1)"
      }
      else if (this.top_level_navigation == 'info') {
        // Info
        return "#1E88E5"
      }
      else 
        return "#1E88E5"
    }
  },
  methods: {
    navigation (value) {
      this.top_level_navigation = value
      if (this.top_level_navigation == 'sports')
        this.page_title = "Scores"
      if (this.top_level_navigation == 'info')
        this.page_title = "Information"
    },
    mlbDataFetch() {
      if (this.$store.state.local) {
        var json = require('../src/assets/json_information/mlb.json'); //(with path)
        this.$store.commit('updating_mlb',json)
        this.$store.commit('api_connected',true)
      }
      else {
      fetch("http://localhost:3000/mlb_game", {
        method: 'GET',
      })
        .then(async response => {
          if(!this.$store.state.local) {
          const data = await response.json();
          this.$store.commit('updating_mlb',data[0])
          console.log(this.$store.state.mlb_game)
          this.$store.commit('api_connected',true)
          // check for error response
          if (!response.ok) {
            // get error message from body or default to response statusText
            const error = (data && data.message) || response.statusText;
            return Promise.reject(error);
          }
          }
        })
        .catch(error => {
          if(!this.$store.state.local) {
          this.$store.commit('api_connected',false)
          console.error("There was an error, auto connection failed!", error);
          }
        });
      }
    },
    nbaDataFetch() {
      if (this.$store.state.local) {
        var json = require('../src/assets/json_information/nba.json'); //(with path)
        this.$store.commit('updating_nba',json)
      }
      else {
      fetch("http://localhost:3000/nba_game", {
        method: 'GET',
      })
        .then(async response => {
          if(!this.$store.state.local) {
          const data = await response.json();
          this.$store.commit('updating_nba',data[0])
          console.log(this.$store.state.nba_game)
          this.$store.commit('api_connected',true)
          // check for error response
          if (!response.ok) {
            // get error message from body or default to response statusText
            const error = (data && data.message) || response.statusText;
            return Promise.reject(error);
          }
          }
        })
        .catch(error => {
          if(!this.$store.state.local) {
          this.$store.commit('api_connected',false)
          console.error("There was an error, auto connection failed!", error);
          }
        });
    }
    } 
  },
  created() {
    this.mlbDataFetch();
    this.nbaDataFetch();
  }
};
</script>
<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Maven+Pro:wght@500&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@700&display=swap');

// This alters Vuetifys Light Theme
.theme--light.v-bottom-navigation .v-btn:not(.v-btn--active) {
    color: rgba(0, 0, 0, 0.3) !important;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .3s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
.v-toolbar__content {
  display: block !important;
}
.v-snack__content {
  text-align: -webkit-center !important;
}
</style>
<style scoped lang="scss">

// Using Maven Pro --> Similar to ars-maquette
.app_title {
  font-family: 'Nunito', sans-serif;
  color: black;
}

.app_title_button {
  margin: 0px !important;
}

.app_title_button_icon{
   transition: all .2s ease-in-out;
}

.app_title_button:active {
  .app_title_button_icon{
    transform: scale(0.75);
  }
}

.spin_icon {
  animation-name: spin;
  animation-duration: 1000ms;
  animation-timing-function: linear; 
}

@keyframes spin {
    from {
        transform:rotate(0deg);
    }
    to {
        transform:rotate(360deg);
    }
}

#no-background-hover::before {
   background-color: transparent !important;
}

.baseball_background {
  margin-bottom: 56px;
  background: linear-gradient(180deg, rgba(170,10,10,1) 0%, rgba(252,170,170,1) 27%, rgba(247,247,247,1) 67%, rgba(235,235,235,1) 100%);
}
.basketball_background {
  margin-bottom: 56px;
  background: linear-gradient(180deg, rgba(255,132,0,1) 0%, rgba(224,146,0,1) 27%, rgba(235,235,235,1) 67%, rgba(235,235,235,1) 100%);
}
.info_background {
  margin-bottom: 56px;
  background: linear-gradient(180deg, rgba(0,168,255,1) 0%, rgba(0,140,213,1) 27%, rgba(235,235,235,1) 67%, rgba(235,235,235,1) 100%);
}

</style>