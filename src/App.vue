<template>
  <div id="app">
      <md-theme md-name="grey">
          <md-toolbar v-if="isTopPanelShown">
              <md-button class="md-icon-button" v-on:click="gotochats">
                  <md-icon >keyboard_backspace</md-icon>
              </md-button>
            <h1 class="md-title">
                <md-input-container>
                    <label>{{ partnerName }}</label>
                    <md-input :value="userDisplayName" @change="setUserName"></md-input>
                </md-input-container>
                </h1>
          </md-toolbar>
    <main>
        <router-view></router-view>
    </main>
      <footer :style="footerCss">
          <div class="bottom-fixed">
              <md-bottom-bar v-if="logged && isBottomPanelShown">
                  <md-bottom-bar-item md-icon="account_balance_wallet" v-on:click="$router.push('/home/')" :md-active="!!$router.currentRoute.path.match(/\/home\//) || !!$router.currentRoute.path.match(/\/transactions\//) || !!$router.currentRoute.path.match(/\/transfer\//)">{{$t('bottom.wallet_button')}}</md-bottom-bar-item>
                  <md-bottom-bar-item md-icon="forum" v-on:click="$router.push('/chats/')" :md-active="!!$router.currentRoute.path.match(/\/chats\//)">{{$t('bottom.chats_button')}}<div class="new-icon" v-if="totalNew">{{ totalNew }}</div></md-bottom-bar-item>
                  <md-bottom-bar-item md-icon="settings" v-on:click="$router.push('/options/')" :md-active="!!$router.currentRoute.path.match(/\/options\//)">{{$t('bottom.settings_button')}}</md-bottom-bar-item>
                  <md-bottom-bar-item md-icon="exit_to_app" v-on:click="exitme" :title="$t('bottom.exit_button_tooltip')">{{$t('bottom.exit_button')}}</md-bottom-bar-item>
              </md-bottom-bar>
          </div>
      </footer>
          <a name="bottom"></a>
      </md-theme>
      <div class="forbid" v-if="disabled" style="display: table;"><div style="display: table-cell; vertical-align: middle;">{{ $t('login.device_unsupported') }}</div></div>
      <audio ref="messageSound" class="newMessageNotification" id="messageSound" src="/static/sound/bbpro_link.mp3"></audio>

  </div>
</template>

<script>
export default {
  name: 'app',
  mounted: function () {
    setInterval(
      (function (self) {
        return function () {
          if (self.$store.state.totalNewChats > 0 && (self.$store.state.notifyBar)) {
            if (window.notify_amount !== self.$store.state.totalNewChats) {
              if (window.notify_interval) {
                clearInterval(window.notify_interval)
              }
              window.notify_toggle = false
              window.notify_amount = self.$store.state.totalNewChats
              window.notify_interval = setInterval(function () {
                if (window.notify_toggle) {
                  document.title = self.$i18n.t('app_title')
                } else {
                  var newTitle = ''
                  if (window.notify_amount % 10 === 1 && window.notify_amount % 100 !== 11) {
                    newTitle = window.notify_amount + ' ' + self.$i18n.t('new_messages_1')
                  } else if (window.notify_amount % 10 >= 2 && window.notify_amount % 10 <= 4 && (window.notify_amount % 100 < 10 || window.notify_amount % 100 >= 20)) {
                    newTitle = window.notify_amount + ' ' + self.$i18n.t('new_messages_2')
                  } else {
                    newTitle = window.notify_amount + ' ' + self.$i18n.t('new_messages_5')
                  }
                  document.title = newTitle
                }
                window.notify_toggle = !window.notify_toggle
              }, 1000)
            }
          } else {
            if (window.notify_interval) {
              clearInterval(window.notify_interval)
              setTimeout(function () {
                document.title = self.$i18n.t('app_title')
              }, 200)
            }

            if (document.title !== self.$i18n.t('app_title')) {
              setTimeout(function () {
                document.title = self.$i18n.t('app_title')
              }, 200)
            }
          }
          self.updateCurrentValues()
        }
      })(this),
      3000
    )
    window.audio = require('simple-audio')

    if (!this.$store.state.passPhrase && this.$route.path !== '/') {
      this.$router.push('/')
    }
  },
  methods: {
    setUserName (val) {
      this.$store.commit('change_partner_name', val)
    },
    gotochats () {
      this.$store.commit('leave_chat')
      if (history.length > 2) {
        this.$router.back()
      } else {
        this.$router.push('/chats/')
      }
    },
    exitme () {
      this.$store.commit('logout')
      this.$router.push('/')
    }
  },
  computed: {
    footerCss () {
      if (this.$store.state.passPhrase) {
        return 'display:block'
      }
      return 'display:none'
    },
    totalNew () {
      return this.$store.state.totalNewChats
    },
    disabled () {
      return this.$store.state.disabled
    },
    userDisplayName () {
      return this.$store.state.partnerDisplayName
    },
    partnerName () {
      return this.$store.state.partnerName
    },
    isTopPanelShown () {
      return this.$store.state.showPanel
    },
    isBottomPanelShown () {
      return this.$store.state.showBottom
    },
    isAjax () {
      return this.$store.state.ajaxIsOngoing
    },
    logged () {
      return this.$store.state.passPhrase
    }
  },
  data () {
    return {
    }
  }
}
</script>

<style>
body {
  margin: 0;
  min-height: 100%;

}
#app {
    min-height: 100vh;
}
html {
    background-color: #fefefe;
}
body.md-theme-default {
    background-color: #fefefe;
}
footer {
    height: 100px;
}
.login {
    position:relative;
}
.aloader {
    position: fixed!important;
    top: 0;
}
.md-toolbar .md-title
{
    min-width: 70%;
}
.md-toolbar .md-title .md-input-container:after {
    background: none ;
}
.md-toolbar .md-title .md-input-container.md-input-focused:after {
    background-color: rgba(0, 0, 0, 0.12);
}
.md-toolbar .md-title .md-input-container
{
    padding-top: 16px;
    min-height: 48px;
}
.md-toolbar .md-title .md-input-container label
{
    top: 13px;
    font-size: 18px;
}
.md-toolbar .md-title .md-input-container.md-input-focused label, .md-toolbar .md-title .md-input-container.md-has-value label {
    top: 0;
    font-size: 12px;
}
.md-bottom-bar .new-icon {
    position: absolute;
    right: -5px;
    top: -30px;
    color: #4A4A4A;
    width: 20px;
    height: 20px;
    background: #BAD3FF;
    border-radius: 10px;
    line-height: 20px;
    text-align: center;
    z-index: 100;
}
.bottom-fixed{
    position:fixed;
    bottom:0;
    width:100%;
    max-width: 800px;
    margin: 0 auto;
    left: 0;
    right: 0;
    z-index: 10;
    background: white;
}

.bottom-fixed .md-bottom-bar {
    box-shadow: none;
    border-top: 1px solid lightgray;
}
.chat, .home, .chats, .settings, .transaction, .transfer, .login {
    max-width: 800px;
    margin: 0 auto;
    margin-top: 25px;
}
.login {
    margin-top:0px;
}
.settings .md-table tbody .md-table-row {
    border:none;
}
.chats .md-list {
    max-width: 90%;
}

.version {
    position:absolute;
    bottom:0;
    right:0;
}
.forbid {
    position:fixed;
    height:100%;
    width:100%;
    top:0;
    left:0;
    background-color: white;
    text-align: center;
    z-index: 100;
    font-size: 2rem;
    line-height: 2rem;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #4A4A4A;
  font-family: 'Exo 2', sans-serif;
    overflow: hidden;
    max-width: 100%;
    height: 100%;

    max-width: 1024px;
    margin-left: auto;
    margin-right: auto;
}
.md-bottom-bar-item.md-active
{
    background-color: #BAD3FF;
    background: repeating-linear-gradient( 140deg, lightgray, lightgray 1px, #FEFEFE 1px, #FEFEFE 15px );
    color: #4A4A4A;
}

main {
  text-align: center;

}

header {
  margin: 0;
  height: 56px;
  padding: 0 16px 0 24px;
  background-color: #484848;
  color: #ffffff;
}
.md-list-item.md-menu-item
{
    background-color:rgba(255,255,255,0.9);
}
.md-primary
{
    background-color: #4A4A4A;
    color: #ffffff;
}
.md-toolbar.md-theme-grey
{
    color: #4A4A4A;
    border-bottom: 1px solid #4A4A4A;
    background-color: #ffffff;
}
header span {
  display: block;
  position: relative;
  font-size: 20px;
  line-height: 1;
  letter-spacing: .02em;
  font-weight: 400;
  box-sizing: border-box;
  padding-top: 16px;
}
@media (max-width: 480px) {
    .version {
        right:8px;
    }
}
</style>
