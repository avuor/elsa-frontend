<template>
  <b-sidebar
    id="sidebar-right"
    right
    no-header
    backdrop
    bg-variant="white"
    class="mobile-menu"
    sidebar-class="mobile-menu"
  >
    <template>
      <b-nav vertical class="main-mobile-nav">
        <b-nav-item class="border-bottom" :to="{ name: 'etusivu' }">
          <font-awesome-icon icon="home" fixed-width size="lg" />
          {{ $t('etusivu') }}
        </b-nav-item>
        <b-nav-item
          v-if="$isErikoistuva()"
          class="border-bottom"
          :to="{ name: 'koulutussuunnitelma' }"
        >
          <font-awesome-icon :icon="['far', 'clipboard']" fixed-width size="lg" />
          {{ $t('koulutussuunnitelma') }}
        </b-nav-item>
        <b-nav-item
          v-if="$isErikoistuva()"
          class="border-bottom"
          :to="{ name: 'tyoskentelyjaksot' }"
        >
          <font-awesome-icon :icon="['far', 'hospital']" fixed-width size="lg" />
          {{ $t('tyoskentelyjaksot') }}
        </b-nav-item>
        <b-nav-item
          v-if="$isErikoistuva()"
          class="border-bottom"
          :to="{ name: 'suoritemerkinnat' }"
        >
          <font-awesome-icon icon="clipboard-check" fixed-width size="lg" />
          {{ $t('suoritemerkinnat') }}
        </b-nav-item>
        <b-nav-item class="border-bottom" :to="{ name: 'arvioinnit' }">
          <font-awesome-icon icon="award" fixed-width size="lg" />
          {{ $t('arvioinnit') }}
        </b-nav-item>
        <b-nav-item
          v-if="$isErikoistuva() || $isKouluttaja() || $isVastuuhenkilo()"
          class="border-bottom"
          :to="{ name: 'koejakso' }"
        >
          <font-awesome-icon icon="clipboard-check" fixed-width size="lg" />
          {{ $t('koejakso') }}
        </b-nav-item>
        <b-nav-item v-if="$isErikoistuva()" class="border-bottom" :to="{ name: 'asiakirjat' }">
          <font-awesome-icon :icon="['far', 'file-alt']" fixed-width size="lg" />
          {{ $t('asiakirjat') }}
        </b-nav-item>
        <b-nav-item class="border-bottom" :to="{ name: 'viestit' }">
          <font-awesome-icon :icon="['far', 'envelope']" fixed-width size="lg" />
          {{ $t('viestit') }}
        </b-nav-item>
      </b-nav>
      <b-nav class="bg-light font-weight-500" vertical>
        <b-nav-item class="text-nowrap px-3" link-classes="text-dark px-0 py-1" disabled>
          <user-avatar :src-base64="avatar" src-content-type="image/jpeg" :title="title" />
        </b-nav-item>
        <b-nav-item class="ml-5" link-classes="p-0 pt-1 pb-2 pb-2" :to="{ name: 'profiili' }">
          {{ $t('oma-profiilini') }}
        </b-nav-item>
        <b-nav-item @click="logout()" class="ml-5" link-classes="p-0 pt-1 pb-3">
          {{ $t('kirjaudu-ulos') }}
        </b-nav-item>
      </b-nav>
      <b-nav class="font-weight-500 justify-content-center d-flex">
        <b-nav-item
          v-for="locale in locales"
          :key="locale"
          :disabled="currentLocale === locale"
          @click="changeLocale(locale)"
        >
          {{ $t(locale) }}
        </b-nav-item>
      </b-nav>
    </template>
  </b-sidebar>
</template>

<script lang="ts">
  import Vue from 'vue'
  import Avatar from 'vue-avatar'
  import Component from 'vue-class-component'

  import UserAvatar from '@/components/user-avatar/user-avatar.vue'
  import store from '@/store'
  import { getTitleFromAuthorities } from '@/utils/functions'

  @Component({
    components: {
      Avatar,
      UserAvatar
    }
  })
  export default class MobileNav extends Vue {
    get account() {
      return store.getters['auth/account']
    }

    get avatar() {
      if (this.account) {
        return this.account.avatar
      }
      return undefined
    }

    get authorities() {
      if (this.account) {
        return this.account.authorities
      }
      return []
    }

    get title() {
      const value = getTitleFromAuthorities(this.authorities)
      return value ? this.$t(value) : undefined
    }

    get currentLocale() {
      return this.$i18n.locale
    }

    get locales() {
      return Object.keys(this.$i18n.messages)
    }

    async logout() {
      await store.dispatch('auth/logout')
    }

    changeLocale(lang: string) {
      this.$i18n.locale = lang
    }
  }
</script>

<style lang="scss">
  @import '~@/styles/variables';

  .mobile-menu {
    top: auto;

    .b-sidebar-right {
      height: auto;
    }

    .nav-link {
      position: relative;
      padding: 0.75rem;
    }

    .main-mobile-nav {
      .router-link-active {
        &:before {
          content: '';
          position: absolute;
          top: 0;
          left: 0;
          height: 100%;
          border-left: 5px solid $primary;
        }
      }
    }

    .b-sidebar-backdrop {
      top: auto;
    }
  }
</style>
