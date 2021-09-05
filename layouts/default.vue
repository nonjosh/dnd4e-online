<template>
  <v-app dark>
    <v-navigation-drawer
      :mini-variant="miniVariant"
      fixed
      app
      expand-on-hover
      permanent
    >
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
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      items: [
        {
          icon: 'mdi-apps',
          title: 'Main',
          to: '/'
        },
        {
          icon: 'mdi-account-group',
          title: 'Chars',
          to: '/chars'
        }
      ],
      miniVariant: false,
      title: 'DnD4e'
    }
  }
}
</script>
