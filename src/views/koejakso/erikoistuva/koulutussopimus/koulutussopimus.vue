<template>
  <div class="koulutussopimus col-lg-8 px-0">
    <b-breadcrumb :items="items" class="mb-0" />
    <b-container fluid>
      <h1 class="mb-3">{{ $t('koulutussopimus') }}</h1>
      <div v-if="!loading">
        <b-row lg>
          <b-col>
            <b-alert :show="showReturned" variant="danger" class="mt-3">
              <div class="d-flex flex-row">
                <em class="align-middle">
                  <font-awesome-icon :icon="['fas', 'exclamation-circle']" class="mr-2" />
                </em>
                <div>
                  {{ $t('koulutussopimus-tila-palautettu-korjattavaksi') }}
                  <span class="d-block">
                    {{ $t('syy') }}&nbsp;
                    <span class="font-weight-500">{{ korjausehdotus }}</span>
                  </span>
                </div>
              </div>
            </b-alert>
            <div v-if="editable">
              <p>{{ $t('koulutussopimus-ingressi-1') }}</p>
              <p>
                {{ $t('koulutussopimus-ingressi-2') }}
                <b-link :to="{ name: 'koejakso-yleiset-tavoitteet' }">
                  {{ $t('koejakso-tavoitteet-linkki') }}
                </b-link>
              </p>
            </div>
            <b-alert :show="showWaitingForAcceptance" variant="dark" class="mt-3">
              <div class="d-flex flex-row">
                <em class="align-middle">
                  <font-awesome-icon :icon="['fas', 'info-circle']" class="text-muted mr-2" />
                </em>
                <div>{{ $t('koulutussopimus-tila-odottaa-hyvaksyntaa') }}</div>
              </div>
            </b-alert>
            <b-alert variant="success" :show="showAcceptedByEveryone">
              <div class="d-flex flex-row">
                <em class="align-middle">
                  <font-awesome-icon :icon="['fas', 'check-circle']" class="mr-2" />
                </em>
                <div>
                  {{ $t('koulutussopimus-tila-hyvaksytty') }}
                  <span class="d-block">
                    {{ $t('koulutussopimus-tila-hyvaksytty-lisatiedot') }}
                  </span>
                </div>
              </div>
            </b-alert>
          </b-col>
        </b-row>
        <hr />
        <b-row>
          <b-col>
            <erikoistuva-details
              :avatar="account.avatar"
              :name="`${account.firstName} ${account.lastName}`"
              :erikoisala="account.erikoistuvaLaakari.erikoisalaNimi"
              :opiskelijatunnus="account.erikoistuvaLaakari.opiskelijatunnus"
              :syntymaaika="account.erikoistuvaLaakari.syntymaaika"
              :yliopisto="account.erikoistuvaLaakari.yliopisto"
            ></erikoistuva-details>
          </b-col>
        </b-row>
        <hr />
        <b-row>
          <b-col>
            <koulutussopimus-form
              v-if="editable"
              :data="koejaksoData.koulutussopimus"
              :account="account"
              :kouluttajat="kouluttajat"
              :vastuuhenkilot="vastuuhenkilot"
              :yliopistot="yliopistot"
              @saveAndExit="onSaveDraftAndExit"
              @submit="onSubmit"
            ></koulutussopimus-form>

            <koulutussopimus-readonly
              v-if="!editable"
              :data="koejaksoData.koulutussopimus"
            ></koulutussopimus-readonly>
          </b-col>
        </b-row>
      </div>
      <div v-else class="text-center">
        <b-spinner variant="primary" :label="$t('ladataan')" />
      </div>
    </b-container>
  </div>
</template>

<script lang="ts">
  import Component from 'vue-class-component'
  import { Vue } from 'vue-property-decorator'

  import { getKoulutussopimusLomake } from '@/api/erikoistuva'
  import ErikoistuvaDetails from '@/components/erikoistuva-details/erikoistuva-details.vue'
  import store from '@/store'
  import { KoulutussopimusLomake, Koejakso, KoejaksonVaiheButtonStates } from '@/types'
  import { LomakeTilat } from '@/utils/constants'
  import { checkCurrentRouteAndRedirect } from '@/utils/functions'
  import { toastFail, toastSuccess } from '@/utils/toast'
  import KoulutussopimusForm from '@/views/koejakso/erikoistuva/koulutussopimus/koulutussopimus-form.vue'
  import KoulutussopimusReadonly from '@/views/koejakso/erikoistuva/koulutussopimus/koulutussopimus-readonly.vue'

  @Component({
    components: {
      ErikoistuvaDetails,
      KoulutussopimusForm,
      KoulutussopimusReadonly
    }
  })
  export default class ErikoistuvaKoulutussopimus extends Vue {
    items = [
      {
        text: this.$t('etusivu'),
        to: { name: 'etusivu' }
      },
      {
        text: this.$t('koejakso'),
        to: { name: 'koejakso' }
      },
      {
        text: this.$t('koulutussopimus'),
        active: true
      }
    ]
    loading = true
    koulutussopimusLomake: null | KoulutussopimusLomake = null
    vastuuhenkilot = []
    yliopistot = []

    get account() {
      return store.getters['auth/account']
    }

    get editable() {
      switch (this.koejaksoData.koulutusSopimuksenTila) {
        case LomakeTilat.PALAUTETTU_KORJATTAVAKSI:
          return true
        case LomakeTilat.UUSI:
          return true
        case LomakeTilat.TALLENNETTU_KESKENERAISENA:
          return true
      }
      return false
    }

    get showWaitingForAcceptance() {
      return (
        this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.ODOTTAA_HYVAKSYNTAA ||
        this.koejaksoData.koulutusSopimuksenTila ===
          LomakeTilat.ODOTTAA_TOISEN_KOULUTTAJAN_HYVAKSYNTAA ||
        this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.ODOTTAA_VASTUUHENKILON_HYVAKSYNTAA
      )
    }

    get showReturned() {
      return this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.PALAUTETTU_KORJATTAVAKSI
    }

    get showAcceptedByEveryone() {
      return this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.HYVAKSYTTY
    }

    get kouluttajat() {
      return store.getters['erikoistuva/kouluttajat']
    }

    get koejaksoData(): Koejakso {
      return store.getters['erikoistuva/koejakso']
    }

    get korjausehdotus() {
      return this.koejaksoData.koulutussopimus?.korjausehdotus
    }

    setKoejaksoData() {
      if (this.koejaksoData.koulutussopimus) {
        this.koulutussopimusLomake = this.koejaksoData.koulutussopimus
      }

      if (!this.editable) {
        this.$emit('skipRouteExitConfirm', true)
      }
    }

    async onSaveDraftAndExit(
      form: KoulutussopimusLomake,
      buttonStates: KoejaksonVaiheButtonStates
    ) {
      buttonStates.secondaryButtonLoading = true
      this.koulutussopimusLomake = form
      try {
        if (this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.UUSI) {
          await store.dispatch('erikoistuva/postKoulutussopimus', this.koulutussopimusLomake)
        }
        if (this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.TALLENNETTU_KESKENERAISENA) {
          await store.dispatch('erikoistuva/putKoulutussopimus', this.koulutussopimusLomake)
        }

        toastSuccess(this, this.$t('koulutussopimus-tallennettu-onnistuneesti'))
        this.$emit('skipRouteExitConfirm', true)
        checkCurrentRouteAndRedirect(this.$router, '/koejakso')
      } catch (err) {
        toastFail(this, this.$t('koulutussopimuksen-tallennus-epaonnistui'))
      }
      buttonStates.secondaryButtonLoading = false
    }

    async saveNewForm() {
      try {
        await store.dispatch('erikoistuva/postKoulutussopimus', this.koulutussopimusLomake)
        toastSuccess(this, this.$t('koulutussopimus-lisatty-onnistuneesti'))
        this.$emit('skipRouteExitConfirm', true)
      } catch (err) {
        toastFail(this, this.$t('koulutussopimuksen-lisaaminen-epaonnistui'))
      }
    }

    async updateSentForm() {
      try {
        await store.dispatch('erikoistuva/putKoulutussopimus', this.koulutussopimusLomake)
        toastSuccess(this, this.$t('koulutussopimus-lisatty-onnistuneesti'))
        this.$emit('skipRouteExitConfirm', true)
      } catch (err) {
        toastFail(this, this.$t('koulutussopimuksen-lisaaminen-epaonnistui'))
      }
    }

    async onSubmit(form: KoulutussopimusLomake, buttonStates: KoejaksonVaiheButtonStates) {
      buttonStates.primaryButtonLoading = true
      this.koulutussopimusLomake = form
      this.koulutussopimusLomake.lahetetty = true

      if (
        this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.TALLENNETTU_KESKENERAISENA ||
        this.koejaksoData.koulutusSopimuksenTila === LomakeTilat.PALAUTETTU_KORJATTAVAKSI
      ) {
        await this.updateSentForm()
      } else {
        await this.saveNewForm()
      }
      buttonStates.primaryButtonLoading = false
    }

    watch() {
      if (!this.editable) {
        this.$emit('skipRouteExitConfirm', true)
      }
    }

    async mounted() {
      this.loading = true

      if (!this.koejaksoData) {
        await store.dispatch('erikoistuva/getKoejakso')
      }
      await store.dispatch('erikoistuva/getKouluttajat')

      this.setKoejaksoData()

      const koulutussopimusLomake = (await getKoulutussopimusLomake()).data
      this.vastuuhenkilot = koulutussopimusLomake.vastuuhenkilot
      this.yliopistot = koulutussopimusLomake.yliopistot.filter(
        (y: any) => y.nimi !== this.account.erikoistuvaLaakari.yliopisto
      )

      this.loading = false

      if (!this.editable) {
        this.$emit('skipRouteExitConfirm', true)
      }
    }
  }
</script>
