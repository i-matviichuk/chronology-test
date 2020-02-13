<template>
  <v-app id="inspire">
    <v-navigation-drawer
      v-model="drawer"
      app
      :clipped="$vuetify.breakpoint.lgAndUp"
    >
      <v-list dense>
        <template>
          <v-row align="center">
            <!--            <v-col cols="6">-->
            <!--              <v-subheader>-->
            <!--                Chronology tree-->
            <!--              </v-subheader>-->
            <!--            </v-col>-->
            <v-col
              cols="6"
              class="text-center"
            >
              <a
                class="body-2 black--text"
                @click="toRoute('Chronology tree')"
              >Chronology tree</a>
            </v-col>
          </v-row>
          <!--          <v-row-->
          <!--            v-if="item.heading"-->
          <!--            :key="item.heading"-->
          <!--            align="center"-->
          <!--          >-->
          <!--            <v-col cols="6">-->
          <!--              <v-subheader v-if="item.heading">-->
          <!--                {{ item.heading }}-->
          <!--              </v-subheader>-->
          <!--            </v-col>-->
          <!--            <v-col-->
          <!--              cols="6"-->
          <!--              class="text-center"-->
          <!--            >-->
          <!--              <a-->
          <!--                href="#!"-->
          <!--                class="body-2 black&#45;&#45;text"-->
          <!--              >EDIT</a>-->
          <!--            </v-col>-->
          <!--          </v-row>-->
          <!--          <v-list-group-->
          <!--            v-else-if="item.children"-->
          <!--            :key="item.text"-->
          <!--            v-model="item.model"-->
          <!--            :prepend-icon="item.model ? item.icon : item['icon-alt']"-->
          <!--            append-icon=""-->
          <!--          >-->
          <!--            <template v-slot:activator>-->
          <!--              <v-list-item>-->
          <!--                <v-list-item-content>-->
          <!--                  <v-list-item-title>-->
          <!--                    {{ item.text }}-->
          <!--                  </v-list-item-title>-->
          <!--                </v-list-item-content>-->
          <!--              </v-list-item>-->
          <!--            </template>-->
          <!--            <v-list-item-->
          <!--              v-for="(child, i) in item.children"-->
          <!--              :key="i"-->
          <!--            >-->
          <!--              &lt;!&ndash;              @click=""&ndash;&gt;-->
          <!--              <v-list-item-action v-if="child.icon">-->
          <!--                <v-icon>{{ child.icon }}</v-icon>-->
          <!--              </v-list-item-action>-->
          <!--              <v-list-item-content>-->
          <!--                <v-list-item-title>-->
          <!--                  {{ child.text }}-->
          <!--                </v-list-item-title>-->
          <!--              </v-list-item-content>-->
          <!--            </v-list-item>-->
          <!--          </v-list-group>-->
          <!--          <v-list-item-->
          <!--            v-else-->
          <!--            :key="item.text"-->
          <!--          >-->
          <!--            &lt;!&ndash;            @click=""&ndash;&gt;-->
          <!--            <v-list-item-action>-->
          <!--              <v-icon>{{ item.icon }}</v-icon>-->
          <!--            </v-list-item-action>-->
          <!--            <v-list-item-content>-->
          <!--              <v-list-item-title>-->
          <!--                {{ item.text }}-->
          <!--              </v-list-item-title>-->
          <!--            </v-list-item-content>-->
          <!--          </v-list-item>-->
        </template>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar
      :clipped-left="$vuetify.breakpoint.lgAndUp"
      app
      color="blue darken-3"
      dark
    >
      <v-toolbar-title
        style="width: 300px"
        class="ml-0 pl-4"
      >
        <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
        <span class="hidden-sm-and-down">Сhronology</span>
      </v-toolbar-title>
      <v-text-field
        flat
        solo-inverted
        hide-details
        prepend-inner-icon="mdi-search"
        label="Search"
        class="hidden-sm-and-down"
      />
      <v-spacer />
      <v-btn icon>
        <v-icon>mdi-apps</v-icon>
      </v-btn>
      <v-btn icon>
        <v-icon>mdi-bell</v-icon>
      </v-btn>
      <v-btn
        icon
        large
      >
        <v-avatar
          size="32px"
          item
        >
          <v-img
            src="https://cdn.vuetifyjs.com/images/logos/logo.svg"
            alt="Vuetify"
          />
        </v-avatar>
      </v-btn>
    </v-app-bar>
    <v-content>
      <router-view />
      <!--      <diagram />-->
    </v-content>
  </v-app>
</template>

<script>
export default {
  name: 'App',
  // props: {
  //   source: String,
  // },
  data: () => ({
    dialog: false,
    drawer: null,
    items: [
      { icon: 'mdi-contacts', text: 'Contacts' },
      { icon: 'mdi-history', text: 'Frequently contacted' },
      { icon: 'mdi-content_copy', text: 'Duplicates' },
      {
        icon: 'mdi-keyboard_arrow_up',
        'icon-alt': 'mdi-keyboard_arrow_down',
        text: 'Labels',
        model: true,
        children: [
          { icon: 'mdi-add', text: 'Create label' },
        ],
      },
      {
        icon: 'mdi-keyboard_arrow_up',
        'icon-alt': 'mdi-keyboard_arrow_down',
        text: 'More',
        model: false,
        children: [
          { text: 'Import' },
          { text: 'Export' },
          { text: 'Print' },
          { text: 'Undo changes' },
          { text: 'Other contacts' },
        ],
      },
      { icon: 'mdi-settings', text: 'Settings' },
      { icon: 'mdi-chat_bubble', text: 'Send feedback' },
      { icon: 'mdi-help', text: 'Help' },
      { icon: 'mdi-phonelink', text: 'App downloads' },
      { icon: 'mdi-keyboard', text: 'Go to the old version' },
    ],
  }),
  methods: {
    toRoute(name) {
      this.$router.push({ name });
    },
  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
