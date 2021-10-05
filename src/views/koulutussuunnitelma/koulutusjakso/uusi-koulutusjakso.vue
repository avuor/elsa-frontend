<template>
  <div class="lisaa-koulutusjakso">
    <b-breadcrumb :items="items" class="mb-0" />
    <b-container fluid>
      <b-row lg>
        <b-col>
          <div v-if="!loading" class="mb-4">
            <h1>{{ $t('lisaa-koulutusjakso') }}</h1>
            <p>{{ $t('koulutusjakso-lisays-ingressi-1') }}</p>
            <p>{{ $t('koulutusjakso-lisays-ingressi-2') }}</p>
            <hr />
            <koulutusjakso-form @submit="onSubmit" @cancel="onCancel" />
          </div>
          <div v-else class="text-center">
            <b-spinner variant="primary" :label="$t('ladataan')" />
          </div>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script lang="ts">
  import { Component, Mixins } from 'vue-property-decorator'

  import KoulutusjaksoForm from '@/forms/koulutusjakso-form.vue'
  import ConfirmRouteExit from '@/mixins/confirm-route-exit'

  @Component({
    components: {
      KoulutusjaksoForm
    }
  })
  export default class UusiKoulutusjakso extends Mixins(ConfirmRouteExit) {
    items = [
      {
        text: this.$t('etusivu'),
        to: { name: 'etusivu' }
      },
      {
        text: this.$t('koulutussuunnitelma'),
        to: { name: 'koulutussuunnitelma' }
      },
      {
        text: this.$t('lisaa-koulutusjakso'),
        active: true
      }
    ]
    loading = true

    async mounted() {
      this.loading = false
    }

    onSubmit() {
      console.log('todo')
    }

    onCancel() {
      this.$router.push({
        name: 'koulutussuunnitelma'
      })
    }
  }
</script>

<style lang="scss" scoped>
  .lisaa-koulutusjakso {
    max-width: 768px;
  }
</style>
