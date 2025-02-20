<template>
  <div class="muokkaa-tyoskentelyjaksoa">
    <b-breadcrumb :items="items" class="mb-0" />
    <b-container fluid>
      <b-row lg>
        <b-col>
          <h1>{{ $t('muokkaa-tyoskentelyjaksoa') }}</h1>
          <hr />
          <tyoskentelyjakso-form
            v-if="!loading && tyoskentelyjakso"
            :value="tyoskentelyjakso"
            :editing="true"
            :kunnat="tyoskentelyjaksoLomake.kunnat"
            :erikoisalat="tyoskentelyjaksoLomake.erikoisalat"
            :asiakirjat="tyoskentelyjakso.asiakirjat"
            @submit="onSubmit"
            @cancel="onCancel"
          />
          <div v-else class="text-center">
            <b-spinner variant="primary" :label="$t('ladataan')" />
          </div>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator'

  import {
    getTyoskentelyjakso,
    getTyoskentelyjaksoLomake,
    putTyoskentelyjakso
  } from '@/api/erikoistuva'
  import TyoskentelyjaksoForm from '@/forms/tyoskentelyjakso-form.vue'
  import { Tyoskentelyjakso, TyoskentelyjaksoLomake } from '@/types'
  import { ErrorKeys } from '@/utils/constants'
  import { toastFail, toastSuccess } from '@/utils/toast'

  @Component({
    components: {
      TyoskentelyjaksoForm
    }
  })
  export default class MuokkaaTyoskentelyjaksoa extends Vue {
    items = [
      {
        text: this.$t('etusivu'),
        to: { name: 'etusivu' }
      },
      {
        text: this.$t('tyoskentelyjaksot'),
        to: { name: 'tyoskentelyjaksot' }
      },
      {
        text: this.$t('muokkaa-tyoskentelyjaksoa'),
        active: true
      }
    ]
    tyoskentelyjaksoLomake: null | TyoskentelyjaksoLomake = null
    tyoskentelyjakso: null | Tyoskentelyjakso = null
    loading = true

    async mounted() {
      await this.fetchTyoskentelyjakso()
      await this.fetchLomake()
      this.loading = false
    }

    async fetchLomake() {
      try {
        this.tyoskentelyjaksoLomake = (await getTyoskentelyjaksoLomake()).data
      } catch (err) {
        toastFail(this, this.$t('tyoskentelyjakson-lomakkeen-hakeminen-epaonnistui'))
      }
    }

    async fetchTyoskentelyjakso() {
      const tyoskentelyjaksoId = this.$route?.params?.tyoskentelyjaksoId
      if (tyoskentelyjaksoId) {
        try {
          this.tyoskentelyjakso = (await getTyoskentelyjakso(tyoskentelyjaksoId)).data
        } catch (err) {
          this.$router.replace({ name: 'tyoskentelyjaksot' })
        }
      }
    }

    async onSubmit(data: any, params: any) {
      params.saving = true
      try {
        const formData = new FormData()

        formData.append('tyoskentelyjaksoJson', JSON.stringify(data.tyoskentelyjakso))
        data.addedFiles.forEach((file: File) => formData.append('files', file, file.name))
        formData.append('deletedAsiakirjaIdsJson', JSON.stringify(data.deletedAsiakirjaIds))

        await putTyoskentelyjakso(formData)

        toastSuccess(this, this.$t('tyoskentelyjakson-tallentaminen-onnistui'))
        this.$emit('skipRouteExitConfirm', true)
        this.$router.push({
          name: 'tyoskentelyjakso',
          params: {
            tyoskentelyjaksoId: `${this.tyoskentelyjakso?.id}`
          }
        })
      } catch (err) {
        if (err.response.data.errorKey === ErrorKeys.TYOSKENTELYAIKA) {
          toastFail(
            this,
            `${this.$t('tyoskentelyjakson-tallentaminen-epaonnistui')}: ${this.$t(
              'tyoskentelyjaksojen-yhteenlaskettu-aika-ylittyy'
            )}`
          )
        } else {
          toastFail(this, this.$t('tyoskentelyjakson-tallentaminen-epaonnistui'))
        }
      }
      params.saving = false
    }

    onCancel() {
      this.$router.push({
        name: 'tyoskentelyjaksot'
      })
    }
  }
</script>

<style lang="scss" scoped>
  .muokkaa-tyoskentelyjaksoa {
    max-width: 768px;
  }
</style>
