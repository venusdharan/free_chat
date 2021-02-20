<template>
  <v-app dark>
    <v-navigation-drawer
      v-model="drawer"
style="width:25%;"
     temporary
      app
    >
    <v-card>
      <v-img
      src="https://cdn.vuetifyjs.com/images/cards/sunshine.jpg"
      height="200px"
    ></v-img>
      <v-card-title>Free Chat</v-card-title>
    </v-card>
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar
      :clipped-left="clipped"
      fixed
      app
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />     
      <v-toolbar-title v-text="title" />
      <v-spacer />
     
    </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    </v-main>
    <v-footer
      :absolute="!fixed"
      app
    >
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Send',
          to: '/send'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Receive',
          to: '/receive'
        }
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Free Chat'
    }
  }
}
</script>
