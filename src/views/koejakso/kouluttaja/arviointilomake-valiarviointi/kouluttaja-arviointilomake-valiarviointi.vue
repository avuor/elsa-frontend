<template>
  <div class="col-lg-8 px-0">
    <b-breadcrumb :items="items" class="mb-0" />
    <b-container fluid>
      <h1 class="mb-3">{{ $t('valiarviointi-kouluttaja') }}</h1>
      <div v-if="!loading">
        <div v-if="editableForEsimies">
          <p>{{ $t('valiarviointi-esimies-ingressi') }}</p>
        </div>
        <div v-else-if="editable">
          <p>{{ $t('valiarviointi-kouluttaja-ingressi') }}</p>
        </div>

        <b-alert :show="showWaitingForLahiesimiesOrErikoistuva" variant="dark" class="mt-3">
          <div class="d-flex flex-row">
            <em class="align-middle">
              <font-awesome-icon :icon="['fas', 'info-circle']" class="text-muted mr-2" />
            </em>
            <div>
              {{ $t('valiarviointi-kouluttaja-allekirjoitettu') }}
            </div>
          </div>
        </b-alert>

        <b-alert :show="showWaitingForErikoistuva" variant="dark" class="mt-3">
          <div class="d-flex flex-row">
            <em class="align-middle">
              <font-awesome-icon :icon="['fas', 'info-circle']" class="text-muted mr-2" />
            </em>
            <div>
              {{ $t('valiarviointi-esimies-allekirjoitettu') }}
            </div>
          </div>
        </b-alert>

        <b-alert
          :show="returned"
          :variant="isCurrentUserLahiesimies ? 'dark' : 'danger'"
          class="mt-3"
        >
          <div class="d-flex flex-row">
            <em class="align-middle">
              <font-awesome-icon
                v-if="isCurrentUserLahiesimies"
                :icon="['fas', 'info-circle']"
                class="text-muted mr-2"
              />
              <font-awesome-icon v-else :icon="['fas', 'exclamation-circle']" class="mr-2" />
            </em>
            <div>
              <span v-if="isCurrentUserLahiesimies">
                {{ $t('valiarviointi-palautettu-kouluttajalle-muokattavaksi-esimies') }}
              </span>
              <span v-else>
                {{ $t('valiarviointi-palautettu-kouluttajalle-muokattavaksi-kouluttaja') }}
              </span>
              <span class="d-block">{{ $t('syy') }} {{ valiarviointi.korjausehdotus }}</span>
            </div>
          </div>
        </b-alert>

        <b-alert variant="success" :show="acceptedByEveryone">
          <div class="d-flex flex-row">
            <em class="align-middle">
              <font-awesome-icon :icon="['fas', 'check-circle']" class="mr-2" />
            </em>
            <span>{{ $t('valiarviointi-tila-hyvaksytty') }}</span>
          </div>
        </b-alert>

        <hr />

        <erikoistuva-details
          :avatar="erikoistuvanAvatar"
          :name="erikoistuvanNimi"
          :erikoisala="valiarviointi.erikoistuvanErikoisala"
          :opiskelijatunnus="valiarviointi.erikoistuvanOpiskelijatunnus"
          :yliopisto="valiarviointi.erikoistuvanYliopisto"
          :show-birthdate="false"
        />

        <hr />

        <div>
          <h3 class="mb-3">{{ $t('soveltuvuus-erikoisalalle-valiarvioinnin-perusteella') }}</h3>
          <b-row>
            <b-col lg="8">
              <div v-if="!editable">
                <h5>{{ $t('edistyminen-osaamistavoitteiden-mukaista') }}</h5>
                <p>
                  {{
                    valiarviointi.edistyminenTavoitteidenMukaista
                      ? $t('kylla')
                      : $t('ei-huolenaiheita-on')
                  }}
                </p>
              </div>
              <elsa-form-group
                v-else
                :label="$t('edistyminen-osaamistavoitteiden-mukaista')"
                :required="true"
              >
                <template v-slot="{ uid }">
                  <b-form-radio-group
                    :id="uid"
                    v-model="valiarviointi.edistyminenTavoitteidenMukaista"
                    :options="edistyminenVaihtoehdot"
                    :state="validateState('edistyminenTavoitteidenMukaista')"
                    stacked
                  ></b-form-radio-group>
                  <b-form-invalid-feedback
                    :id="`${uid}-feedback`"
                    :state="validateState('edistyminenTavoitteidenMukaista')"
                  >
                    {{ $t('pakollinen-tieto') }}
                  </b-form-invalid-feedback>
                </template>
              </elsa-form-group>
            </b-col>
          </b-row>
          <b-row v-if="valiarviointi.edistyminenTavoitteidenMukaista === false">
            <b-col lg="8">
              <div v-if="!editable">
                <h5>{{ $t('keskustelu-ja-toimenpiteet-tarpeen-ennen-hyvaksymista') }}</h5>
                <ul class="pl-4">
                  <li v-for="kategoria in sortedKategoriat" :key="kategoria">
                    {{ naytaKehittamistoimenpideKategoria(kategoria) }}
                  </li>
                </ul>
              </div>
              <elsa-form-group
                v-else
                :label="$t('keskustelu-ja-toimenpiteet-tarpeen-ennen-hyvaksymista')"
              >
                <template v-slot="{ uid }">
                  <b-form-checkbox-group
                    :id="uid"
                    v-model="valiarviointi.kehittamistoimenpideKategoriat"
                    :options="kehittamistoimenpideKategoriat"
                    :required="true"
                    :state="validateState('kehittamistoimenpideKategoriat')"
                    stacked
                  ></b-form-checkbox-group>
                  <b-form-invalid-feedback
                    :id="`${uid}-feedback`"
                    :state="validateState('kehittamistoimenpideKategoriat')"
                  >
                    {{ $t('pakollinen-tieto') }}
                  </b-form-invalid-feedback>
                  <div class="ml-4">
                    <b-form-input
                      v-if="muuValittu"
                      v-model="valiarviointi.muuKategoria"
                      :state="validateState('muuKategoria')"
                    ></b-form-input>
                    <b-form-invalid-feedback>{{ $t('pakollinen-tieto') }}</b-form-invalid-feedback>
                  </div>
                </template>
              </elsa-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <div v-if="!editable">
                <div v-if="valiarviointi.vahvuudet">
                  <h5>{{ $t('vahvuudet') }}</h5>
                  <p v-if="!editable">{{ valiarviointi.vahvuudet }}</p>
                </div>
              </div>
              <elsa-form-group v-else :label="$t('vahvuudet')">
                <template v-slot="{ uid }">
                  <b-form-textarea
                    :id="uid"
                    v-model="valiarviointi.vahvuudet"
                    rows="7"
                    class="textarea-min-height"
                  ></b-form-textarea>
                  <b-form-invalid-feedback :id="`${uid}-feedback`">
                    {{ $t('pakollinen-tieto') }}
                  </b-form-invalid-feedback>
                </template>
              </elsa-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <div v-if="!editable">
                <div v-if="valiarviointi.kehittamistoimenpiteet">
                  <h5>{{ $t('selvitys-kehittamistoimenpiteista') }}</h5>
                  <p v-if="!editable">{{ valiarviointi.kehittamistoimenpiteet }}</p>
                </div>
              </div>
              <elsa-form-group v-else :label="$t('selvitys-kehittamistoimenpiteista')">
                <template v-slot="{ uid }">
                  <b-form-textarea
                    :id="uid"
                    v-model="valiarviointi.kehittamistoimenpiteet"
                    rows="7"
                    class="textarea-min-height"
                  ></b-form-textarea>
                  <b-form-invalid-feedback :id="`${uid}-feedback`">
                    {{ $t('pakollinen-tieto') }}
                  </b-form-invalid-feedback>
                </template>
              </elsa-form-group>
            </b-col>
          </b-row>
          <hr />
        </div>
        <koulutuspaikan-arvioijat
          :lahikouluttaja="valiarviointi.lahikouluttaja"
          :lahiesimies="valiarviointi.lahiesimies"
          :isReadonly="true"
        />
        <hr />

        <div v-if="allekirjoitukset.length > 0">
          <koejakson-vaihe-allekirjoitukset :allekirjoitukset="allekirjoitukset" />
        </div>

        <div v-if="editable || editableForEsimies">
          <hr v-if="allekirjoitukset.length > 0" />
          <b-row>
            <b-col class="text-right">
              <elsa-button
                class="ml-1 mr-3 d-block d-md-inline-block d-lg-block d-xl-inline-block text-left"
                style="max-width: 14rem"
                variant="back"
                :to="{ name: 'koejakso' }"
              >
                {{ $t('peruuta') }}
              </elsa-button>
              <elsa-button
                v-if="isCurrentUserLahiesimies"
                class="my-2 mr-3 d-block d-md-inline-block d-lg-block d-xl-inline-block"
                style="min-width: 14rem"
                variant="outline-primary"
                :disabled="buttonStates.primaryButtonLoading"
                :loading="buttonStates.secondaryButtonLoading"
                v-b-modal.return-to-sender
              >
                {{ $t('palauta-muokattavaksi') }}
              </elsa-button>
              <elsa-button
                class="my-2 mr-3 d-block d-md-inline-block d-lg-block d-xl-inline-block"
                style="min-width: 14rem"
                variant="primary"
                :disabled="buttonStates.secondaryButtonLoading"
                :loading="buttonStates.primaryButtonLoading"
                @click="onValidateAndConfirm"
              >
                {{ $t('allekirjoita-laheta') }}
              </elsa-button>
            </b-col>
          </b-row>
        </div>
      </div>
      <div v-else class="text-center">
        <b-spinner variant="primary" :label="$t('ladataan')" />
      </div>
    </b-container>

    <elsa-confirmation-modal
      id="confirm-send"
      :title="$t('vahvista-lomakkeen-lahetys')"
      :text="
        isCurrentUserLahiesimies
          ? $t('vahvista-koejakson-vaihe-erikoistuvalle')
          : $t('vahvista-koejakson-vaihe-esimiehelle')
      "
      :submitText="$t('allekirjoita-laheta')"
      @submit="onSign"
    />

    <elsa-return-to-sender-modal
      id="return-to-sender"
      :title="$t('palauta-kouluttajalle-muokattavaksi')"
      @submit="onReturnToSender"
    />
  </div>
</template>

<script lang="ts">
  import _get from 'lodash/get'
  import Component from 'vue-class-component'
  import { Mixins } from 'vue-property-decorator'
  import { validationMixin } from 'vuelidate'
  import { required, requiredIf } from 'vuelidate/lib/validators'

  import { getValiarviointi } from '@/api/kouluttaja'
  import ElsaButton from '@/components/button/button.vue'
  import ErikoistuvaDetails from '@/components/erikoistuva-details/erikoistuva-details.vue'
  import ElsaFormGroup from '@/components/form-group/form-group.vue'
  import KoejaksonVaiheAllekirjoitukset from '@/components/koejakson-vaiheet/koejakson-vaihe-allekirjoitukset.vue'
  import KoulutuspaikanArvioijat from '@/components/koejakson-vaiheet/koulutuspaikan-arvioijat.vue'
  import ElsaConfirmationModal from '@/components/modal/confirmation-modal.vue'
  import ElsaReturnToSenderModal from '@/components/modal/return-to-sender-modal.vue'
  import store from '@/store'
  import {
    ValiarviointiLomake,
    KoejaksonVaiheButtonStates,
    KoejaksonVaiheAllekirjoitus
  } from '@/types'
  import { KehittamistoimenpideKategoria, LomakeTilat } from '@/utils/constants'
  import { checkCurrentRouteAndRedirect } from '@/utils/functions'
  import * as allekirjoituksetHelper from '@/utils/koejaksonVaiheAllekirjoitusMapper'
  import { toastFail, toastSuccess } from '@/utils/toast'

  @Component({
    components: {
      ErikoistuvaDetails,
      ElsaFormGroup,
      ElsaButton,
      ElsaConfirmationModal,
      ElsaReturnToSenderModal,
      KoulutuspaikanArvioijat,
      KoejaksonVaiheAllekirjoitukset
    },
    validations: {
      valiarviointi: {
        edistyminenTavoitteidenMukaista: {
          required
        },
        kehittamistoimenpideKategoriat: {
          required: requiredIf((value) => {
            return value.edistyminenTavoitteidenMukaista === false
          })
        },
        muuKategoria: {
          required: requiredIf((value) => {
            return value.kehittamistoimenpideKategoriat?.includes(KehittamistoimenpideKategoria.MUU)
          })
        }
      }
    }
  })
  export default class KouluttajaArviointilomakeValiarviointi extends Mixins(validationMixin) {
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
        text: this.$t('valiarviointi-kouluttaja'),
        active: true
      }
    ]

    edistyminenVaihtoehdot = [
      {
        text: this.$t('kylla'),
        value: true
      },
      {
        text: this.$t('ei-huolenaiheita-on'),
        value: false
      }
    ]

    buttonStates: KoejaksonVaiheButtonStates = {
      primaryButtonLoading: false,
      secondaryButtonLoading: false
    }

    loading = true

    kategoriaOrder = [
      KehittamistoimenpideKategoria.TYOSSASUORIUTUMINEN,
      KehittamistoimenpideKategoria.TYOKAYTTAYTYMINEN,
      KehittamistoimenpideKategoria.POTILASPALAUTE,
      KehittamistoimenpideKategoria.MUU
    ]

    valiarviointi: ValiarviointiLomake | null = null

    validateState(value: string) {
      const form = this.$v.valiarviointi
      const { $dirty, $error } = _get(form, value) as any
      return $dirty ? ($error ? false : null) : null
    }

    get valiarvioinninTila() {
      return store.getters['kouluttaja/koejaksot'].find((k: any) => k.id === this.valiarviointi?.id)
        ?.tila
    }

    get valiarviointiId() {
      return Number(this.$route.params.id)
    }

    get editable() {
      return !this.isCurrentUserLahiesimies && !this.valiarviointi?.lahikouluttaja.sopimusHyvaksytty
    }

    get editableForEsimies() {
      return (
        this.isCurrentUserLahiesimies &&
        this.valiarviointi?.lahikouluttaja.sopimusHyvaksytty &&
        !this.valiarviointi?.lahiesimies.sopimusHyvaksytty
      )
    }

    get isCurrentUserLahiesimies() {
      const currentUser = store.getters['auth/account']
      return this.valiarviointi?.lahiesimies.kayttajaUserId === currentUser.id
    }

    get erikoistuvanAvatar() {
      return this.valiarviointi?.erikoistuvanAvatar
    }

    get erikoistuvanNimi() {
      return this.valiarviointi?.erikoistuvanNimi
    }

    get kehittamistoimenpideKategoriat() {
      return Object.keys(KehittamistoimenpideKategoria).map((k) => ({
        text: this.$t('kehittamistoimenpidekategoria-' + k),
        value: k
      }))
    }

    naytaKehittamistoimenpideKategoria(kategoria: string) {
      if (kategoria === KehittamistoimenpideKategoria.MUU) return this.valiarviointi?.muuKategoria
      return this.$t('kehittamistoimenpidekategoria-' + kategoria)
    }

    get muuValittu() {
      return this.valiarviointi?.kehittamistoimenpideKategoriat?.includes(
        KehittamistoimenpideKategoria.MUU
      )
    }

    get sortedKategoriat() {
      return this.valiarviointi?.kehittamistoimenpideKategoriat?.sort(
        (a, b) => this.kategoriaOrder.indexOf(a) - this.kategoriaOrder.indexOf(b)
      )
    }

    get showWaitingForLahiesimiesOrErikoistuva() {
      return (
        !this.isCurrentUserLahiesimies &&
        this.valiarviointi?.lahikouluttaja.sopimusHyvaksytty &&
        !this.valiarviointi?.erikoistuvaAllekirjoittanut
      )
    }

    get showWaitingForErikoistuva() {
      return (
        this.isCurrentUserLahiesimies &&
        this.valiarviointi?.lahiesimies.sopimusHyvaksytty &&
        !this.valiarviointi?.erikoistuvaAllekirjoittanut
      )
    }

    get acceptedByEveryone() {
      return this.valiarviointi?.erikoistuvaAllekirjoittanut
    }

    get returned() {
      return this.valiarvioinninTila === LomakeTilat.PALAUTETTU_KORJATTAVAKSI
    }

    get allekirjoitukset(): KoejaksonVaiheAllekirjoitus[] {
      const allekirjoitusErikoistuva = allekirjoituksetHelper.mapAllekirjoitusErikoistuva(
        this,
        this.valiarviointi?.erikoistuvanNimi,
        this.valiarviointi?.erikoistuvanAllekirjoitusaika
      )
      const allekirjoitusLahikouluttaja = allekirjoituksetHelper.mapAllekirjoitusLahikouluttaja(
        this,
        this.valiarviointi?.lahikouluttaja
      )
      const allekirjoitusLahiesimies = allekirjoituksetHelper.mapAllekirjoitusLahiesimies(
        this,
        this.valiarviointi?.lahiesimies
      )

      return [
        allekirjoitusLahikouluttaja,
        allekirjoitusLahiesimies,
        allekirjoitusErikoistuva
      ].filter((a): a is KoejaksonVaiheAllekirjoitus => a !== null)
    }

    async onReturnToSender(korjausehdotus: string) {
      const form = {
        ...this.valiarviointi,
        korjausehdotus: korjausehdotus,
        lahetetty: false
      }

      try {
        this.buttonStates.secondaryButtonLoading = true
        await store.dispatch('kouluttaja/putValiarviointi', form)
        this.buttonStates.secondaryButtonLoading = false
        checkCurrentRouteAndRedirect(this.$router, '/koejakso')
        toastSuccess(this, this.$t('valiarviointi-palautettu-muokattavaksi'))
      } catch (err) {
        toastFail(this, this.$t('valiarviointi-palautus-epaonnistui'))
      }
    }

    async onSign() {
      if (!this.valiarviointi) return
      if (this.valiarviointi.edistyminenTavoitteidenMukaista === true) {
        this.valiarviointi.kehittamistoimenpideKategoriat = null
      }
      if (
        !this.valiarviointi.kehittamistoimenpideKategoriat?.includes(
          KehittamistoimenpideKategoria.MUU
        )
      ) {
        this.valiarviointi.muuKategoria = null
      }
      try {
        this.buttonStates.primaryButtonLoading = true
        await store.dispatch('kouluttaja/putValiarviointi', this.valiarviointi)
        this.buttonStates.primaryButtonLoading = false
        checkCurrentRouteAndRedirect(this.$router, '/koejakso')
        toastSuccess(this, this.$t('valiarviointi-allekirjoitettu-ja-lahetetty-onnistuneesti'))
      } catch (err) {
        toastFail(this, this.$t('valiarvioinnin-tallennus-epaonnistui'))
      }
    }

    onValidateAndConfirm() {
      if (!this.isCurrentUserLahiesimies) {
        this.$v.$touch()
        if (this.$v.$anyError) {
          return
        }
      }
      return this.$bvModal.show('confirm-send')
    }

    async mounted() {
      this.loading = true
      await store.dispatch('kouluttaja/getKoejaksot')
      const { data } = await getValiarviointi(this.valiarviointiId)
      this.valiarviointi = data
      this.loading = false
    }
  }
</script>

<style lang="scss">
  .textarea-min-height {
    min-height: 100px;
  }

  .custom-control.custom-checkbox {
    padding-top: 0.25rem;
    padding-bottom: 0.25rem;
  }
</style>
