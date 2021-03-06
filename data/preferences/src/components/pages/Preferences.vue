<template>
  <div>
    <h1>General</h1>

    <table>
      <tr>
        <td>
          <label for="hotkey-show-app">Hotkey</label>
        </td>
        <td>
          <b-form-input
            id="hotkey-show-app"
            @focus.native="showHotkeyDialog($event)"
            v-model="hotkey_show_app"></b-form-input>
        </td>
      </tr>

      <tr>
        <td>
          <label for="autostart">Launch at Login</label>
        </td>
        <td>
          <b-form-checkbox
            :disabled="!autostart_allowed"
            id="autostart"
            @change="updateLaunchAtLogin"
            v-model="autostart_enabled"></b-form-checkbox>
        </td>
      </tr>

      <tr>
        <td>
          <label for="show-indicator-icon">Show Indicator Icon</label>
        </td>
        <td>
          <b-form-checkbox
            id="show-indicator-icon"
            @change="updateShowIndicatorIcon"
            v-model="show_indicator_icon"></b-form-checkbox>
        </td>
      </tr>

      <tr>
        <td>
          <label for="show-recent-apps">Show Frequent Apps</label>
        </td>
        <td>
          <b-form-checkbox
            id="show-recent-apps"
            @change="updateShowRecentApps"
            v-model="show_recent_apps"></b-form-checkbox>
        </td>
      </tr>

      <tr>
        <td>
          <label for="theme-name">Theme</label>
        </td>
        <td>
          <b-form-radio
            id="theme-name"
            :options="theme_options"
            @change.native="updateTheme"
            v-model="theme_name"></b-form-radio>
        </td>
      </tr>
    </table>

    <h1>Advanced</h1>

    <table>
      <tr>
        <td>
          <label for="clear_previous_query">Clear Query <br> When App Looses Focus</label>
        </td>
        <td>
          <b-form-checkbox
            id="clear_previous_query"
            @change="updateClearPrevText"
            v-model="clear_previous_query"></b-form-checkbox>
        </td>
      </tr>

      <tr>
        <td class="pull-top">
          <label>Blacklisted app dirs</label>
          <small>
            <p>
              Ulauncher won't search for .desktop files in these dirs
            </p>
            <p v-if="blacklistedDirsChanged">
              <i class="fa fa-warning"></i> Restart Ulauncher for this to take effect
            </p>
          </small>
        </td>
        <td class="pull-top">
          <editable-text-list
            width="450px"
            v-model="blacklisted_desktop_dirs"
            @input="updateBlacklistedDirs"
            newItemPlaceholder="Type in an absolute path and press Enter"></editable-text-list>
        </td>
      </tr>

      </table>
  </div>
</template>

<script>
import jsonp from '@/api'
import bus from '@/event-bus'
import EditableTextList from '@/components/widgets/EditableTextList'

const hotkeyEventName = 'hotkey-show-app'

export default {
  name: 'preferences',

  components: {
    'editable-text-list': EditableTextList
  },

  created () {
    this.fetchData()
    bus.$on(hotkeyEventName, this.onHotkeySet)
  },

  beforeDestroy () {
    bus.$off(hotkeyEventName, this.onHotkeySet)
  },

  data () {
    return {
      hotkey_show_app: '',
      autostart_allowed: false,
      autostart_enabled: false,
      show_recent_apps: false,
      show_indicator_icon: false,
      theme_name: null,
      previous_theme_name: null,
      clear_previous_query: false,
      blacklisted_desktop_dirs: [],
      blacklistedDirsChanged: false,
      theme_options: [
        {text: 'Dark', value: 'dark'},
        {text: 'Light', value: 'light'}
      ]
    }
  },

  methods: {
    fetchData () {
      jsonp('prefs:///get/all').then((data) => {
        this.hotkey_show_app = data.hotkey_show_app
        this.autostart_allowed = data.autostart_allowed
        this.autostart_enabled = data.autostart_enabled
        this.show_recent_apps = data.show_recent_apps
        this.show_indicator_icon = data.show_indicator_icon
        this.theme_name = data.theme_name
        this.clear_previous_query = data.clear_previous_query
        this.blacklisted_desktop_dirs = data.blacklisted_desktop_dirs ? data.blacklisted_desktop_dirs.split(':') : []
      }, (err) => bus.$emit('error', err))
    },

    showHotkeyDialog (e) {
      jsonp('prefs://show/hotkey-dialog', {name: hotkeyEventName})
      e.target.blur()
    },

    onHotkeySet (e) {
      const previous = this.hotkey_show_app
      this.hotkey_show_app = e.displayValue
      jsonp('prefs://set/hotkey-show-app', {value: e.value}).then(null, (err) => {
        this.hotkey_show_app = previous
        bus.$emit('error', err)
      })
    },

    updateLaunchAtLogin () {
      jsonp('prefs://set/autostart-enabled', {value: this.autostart_enabled}).then(null, (err) => {
        this.autostart_enabled = !this.autostart_enabled
        bus.$emit('error', err)
      })
    },

    updateShowIndicatorIcon () {
      jsonp('prefs://set/show-indicator-icon', {value: this.show_indicator_icon}).then(null, (err) => {
        this.show_indicator_icon = !this.show_indicator_icon
        bus.$emit('error', err)
      })
    },

    updateShowRecentApps () {
      jsonp('prefs://set/show-recent-apps', {value: this.show_recent_apps}).then(null, (err) => {
        this.show_recent_apps = !this.show_recent_apps
        bus.$emit('error', err)
      })
    },

    updateTheme (e) {
      jsonp('prefs://set/theme-name', {value: this.theme_name}).then(null, (err) => {
        bus.$emit('error', err)
      })
    },

    updateClearPrevText () {
      jsonp('prefs://set/clear-previous-query', {value: this.clear_previous_query}).then(null, (err) => {
        this.clear_previous_query = !this.clear_previous_query
        bus.$emit('error', err)
      })
    },

    updateBlacklistedDirs () {
      jsonp('prefs://set/blacklisted-desktop-dirs', {value: this.blacklisted_desktop_dirs.join(':')})
        .then(() => {
          this.blacklistedDirsChanged = true
        }, (err) => {
          bus.$emit('error', err)
        })
    }

  }
}
</script>

<style lang="scss" scoped>
/* use tables to support WebKit on Ubuntu 14.04 */
table {
  width: 100%;
  margin: 25px 15px 15px 40px;
}
.pull-top {
  vertical-align: top;
}
h1 {
  margin: 30px 0 0 25px;
  font-size: 110%;
  color: #aaa;
  text-shadow: 1px 1px 1px #fff;
}
td:first-child {
  box-sizing: border-box;
  width: 220px;
  padding-right: 20px;
}
td {padding-bottom: 20px;}
tr:last-child td {padding-bottom: 0;}
label {
  cursor: pointer;

  &+small {
    position: relative;
    top: -5px;
    line-height: 1.3em;
    display: block;
    color: #888;
  }
}
#hotkey-show-app {
  cursor: pointer;
  width: 200px;
}
</style>
