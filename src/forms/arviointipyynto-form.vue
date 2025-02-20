<template>
  <b-form @submit.stop.prevent="onSubmit">
    <elsa-form-group :label="$t('erikoistuva-laakari')">
      <template v-slot="{ uid }">
        <user-avatar :id="uid" :src-base64="avatar" src-content-type="image/jpeg" />
      </template>
    </elsa-form-group>
    <elsa-form-group
      :label="$t('tyoskentelyjakso')"
      :add-new-enabled="true"
      :add-new-label="$t('lisaa-tyoskentelyjakso')"
      :required="true"
      @submit="onTyoskentelyjaksoSubmit"
    >
      <template v-slot:modal-content="{ submit, cancel }">
        <tyoskentelyjakso-form
          :kunnat="kunnat"
          :erikoisalat="erikoisalat"
          @submit="submit"
          @cancel="cancel"
        />
      </template>
      <template v-slot="{ uid }">
        <elsa-form-multiselect
          :id="uid"
          v-model="form.tyoskentelyjakso"
          :options="tyoskentelyjaksotFormatted"
          :state="validateState('tyoskentelyjakso')"
          label="label"
          track-by="id"
          @select="onTyoskentelyjaksoSelect"
        />
        <b-form-invalid-feedback :id="`${uid}-feedback`">
          {{ $t('pakollinen-tieto') }}
        </b-form-invalid-feedback>
      </template>
    </elsa-form-group>
    <elsa-form-group :label="$t('arvioitava-kokonaisuus')" :required="true">
      <template v-slot="{ uid }">
        <elsa-form-multiselect
          :id="uid"
          v-model="form.arvioitavaOsaalue"
          :options="arvioitavanKokonaisuudenKategoriat"
          :state="validateState('arvioitavaOsaalue')"
          group-values="arvioitavatKokonaisuudet"
          group-label="nimi"
          :group-select="false"
          label="nimi"
          track-by="id"
        >
          <template slot="option" slot-scope="props">
            <span v-if="props.option.$isLabel">{{ props.option.$groupLabel }}</span>
            <span v-else class="d-inline-block ml-3">{{ props.option.nimi }}</span>
          </template>
        </elsa-form-multiselect>
        <b-form-invalid-feedback :id="`${uid}-feedback`">
          {{ $t('pakollinen-tieto') }}
        </b-form-invalid-feedback>
      </template>
    </elsa-form-group>
    <elsa-form-group :label="$t('arvioitava-tapahtuma')" :required="true">
      <template v-slot="{ uid }">
        <b-form-input
          :id="uid"
          v-model="form.arvioitavaTapahtuma"
          :state="validateState('arvioitavaTapahtuma')"
          :aria-describedby="`${uid}-feedback`"
        ></b-form-input>
        <b-form-invalid-feedback :id="`${uid}-feedback`">
          {{ $t('pakollinen-tieto') }}
        </b-form-invalid-feedback>
      </template>
    </elsa-form-group>
    <b-form-row>
      <elsa-form-group
        :label="$t('kouluttaja-tai-vastuuhenkilo')"
        :add-new-enabled="!editing"
        :add-new-label="$t('lisaa-kouluttaja')"
        :required="true"
        class="col-md-8"
        @submit="onKouluttajaSubmit"
      >
        <template v-slot:modal-content="{ submit, cancel }">
          <kouluttaja-form @submit="submit" @cancel="cancel" />
        </template>
        <template v-slot="{ uid }">
          <elsa-form-multiselect
            :id="uid"
            v-model="form.arvioinninAntaja"
            :options="kouluttajatAndVastuuhenkilot"
            :state="validateState('arvioinninAntaja')"
            :disabled="editing"
            label="nimi"
            track-by="nimi"
          >
            <template v-slot:option="{ option }">
              <user-avatar
                :src-base64="option.avatar"
                src-content-type="image/jpeg"
                :display-name="option.nimi"
              />
            </template>
          </elsa-form-multiselect>
          <b-form-invalid-feedback :id="`${uid}-feedback`">
            {{ $t('pakollinen-tieto') }}
          </b-form-invalid-feedback>
        </template>
      </elsa-form-group>
      <elsa-form-group :label="$t('ajankohta')" class="col-md-4" :required="true">
        <template v-slot="{ uid }">
          <elsa-form-datepicker
            :id="uid"
            v-model="form.tapahtumanAjankohta"
            :state="validateState('tapahtumanAjankohta')"
            :min="tyoskentelyjaksonAlkamispaiva"
            :max="tyoskentelyjaksonPaattymispaiva"
          ></elsa-form-datepicker>
          <b-form-invalid-feedback :id="`${uid}-feedback`">
            {{ $t('pakollinen-tieto') }}
          </b-form-invalid-feedback>
        </template>
      </elsa-form-group>
    </b-form-row>
    <elsa-form-group :label="$t('lisatiedot')">
      <template v-slot="{ uid }">
        <b-form-textarea :id="uid" v-model="form.lisatiedot" rows="5"></b-form-textarea>
      </template>
    </elsa-form-group>
    <div class="text-right mb-2">
      <elsa-button variant="back" :to="{ name: 'arvioinnit' }">{{ $t('peruuta') }}</elsa-button>
      <elsa-button
        v-if="editing"
        :loading="params.deleting"
        variant="outline-danger"
        class="ml-2"
        @click="onArviointipyyntoDelete"
      >
        {{ $t('poista-arviointipyynto') }}
      </elsa-button>
      <elsa-button :loading="params.saving" type="submit" variant="primary" class="ml-2">
        {{ editing ? $t('tallenna') : $t('laheta') }}
      </elsa-button>
    </div>
  </b-form>
</template>

<script lang="ts">
  import axios from 'axios'
  import Component from 'vue-class-component'
  import { Prop, Mixins } from 'vue-property-decorator'
  import { validationMixin } from 'vuelidate'
  import { required } from 'vuelidate/lib/validators'

  import ElsaButton from '@/components/button/button.vue'
  import ElsaFormDatepicker from '@/components/datepicker/datepicker.vue'
  import ElsaFormGroup from '@/components/form-group/form-group.vue'
  import ElsaFormMultiselect from '@/components/multiselect/multiselect.vue'
  import UserAvatar from '@/components/user-avatar/user-avatar.vue'
  import KouluttajaForm from '@/forms/kouluttaja-form.vue'
  import TyoskentelyjaksoForm from '@/forms/tyoskentelyjakso-form.vue'
  import TyoskentelyjaksoMixin from '@/mixins/tyoskentelyjakso'
  import store from '@/store'
  import {
    ArvioitavanKokonaisuudenKategoria,
    Erikoisala,
    Kayttaja,
    Kunta,
    Suoritusarviointi,
    Tyoskentelyjakso
  } from '@/types'
  import { toastSuccess, toastFail } from '@/utils/toast'
  import { tyoskentelyjaksoLabel } from '@/utils/tyoskentelyjakso'

  @Component({
    components: {
      KouluttajaForm,
      TyoskentelyjaksoForm,
      ElsaFormGroup,
      ElsaFormMultiselect,
      UserAvatar,
      ElsaFormDatepicker,
      ElsaButton
    },
    validations: {
      form: {
        tyoskentelyjakso: {
          required
        },
        arvioitavaOsaalue: {
          required
        },
        arvioitavaTapahtuma: {
          required
        },
        arvioinninAntaja: {
          required
        },
        tapahtumanAjankohta: {
          required
        }
      }
    }
  })
  export default class ArviointipyyntoForm extends Mixins(validationMixin, TyoskentelyjaksoMixin) {
    @Prop({ required: false, default: () => [] })
    tyoskentelyjaksot!: Tyoskentelyjakso[]

    @Prop({ required: false, default: () => [] })
    kunnat!: Kunta[]

    @Prop({ required: false, default: () => [] })
    erikoisalat!: Erikoisala[]

    @Prop({ required: false, default: () => [] })
    arvioitavanKokonaisuudenKategoriat!: ArvioitavanKokonaisuudenKategoria[]

    @Prop({ required: false, default: () => [] })
    kouluttajatAndVastuuhenkilot!: Kayttaja[]

    @Prop({ required: false, default: false })
    editing!: boolean

    @Prop({
      required: false,
      type: Object,
      default: () => ({
        tyoskentelyjakso: null,
        arvioitavaKokonaisuus: null,
        arvioitavaTapahtuma: null,
        kouluttajaOrVastuuhenkilo: null,
        tapahtumanAjankohta: null,
        lisatiedot: null
      })
    })
    value?: Suoritusarviointi

    form: Partial<Suoritusarviointi> = {
      tyoskentelyjakso: null,
      arvioitavaOsaalue: null,
      arvioitavaTapahtuma: null,
      arvioinninAntaja: null,
      tapahtumanAjankohta: null,
      lisatiedot: null
    }
    params = {
      saving: false,
      deleting: false
    }

    mounted() {
      this.form.tyoskentelyjakso = this.value?.tyoskentelyjakso
      if (this.form.tyoskentelyjakso) {
        this.form.tyoskentelyjakso.label = tyoskentelyjaksoLabel(this, this.value?.tyoskentelyjakso)
      }
      this.form.arvioitavaOsaalue = this.value?.arvioitavaOsaalue
      this.form.arvioitavaTapahtuma = this.value?.arvioitavaTapahtuma
      this.form.arvioinninAntaja = this.value?.arvioinninAntaja
      this.form.tapahtumanAjankohta = this.value?.tapahtumanAjankohta
      this.form.lisatiedot = this.value?.lisatiedot
    }

    validateState(name: string) {
      const { $dirty, $error } = this.$v.form[name] as any
      return $dirty ? ($error ? false : null) : null
    }

    onSubmit() {
      this.$v.form.$touch()
      if (this.$v.form.$anyError) {
        return
      }
      this.$emit(
        'submit',
        {
          tyoskentelyjaksoId: this.form.tyoskentelyjakso?.id,
          arvioitavaOsaalueId: this.form.arvioitavaOsaalue?.id,
          arvioitavaTapahtuma: this.form.arvioitavaTapahtuma,
          arvioinninAntajaId: this.form.arvioinninAntaja?.id,
          tapahtumanAjankohta: this.form.tapahtumanAjankohta,
          lisatiedot: this.form.lisatiedot
        },
        this.params
      )
    }

    onArviointipyyntoDelete() {
      this.$emit('delete', this.params)
    }

    async onKouluttajaSubmit(value: Kayttaja, params: any) {
      params.saving = true
      try {
        const kouluttaja = (await axios.post('/erikoistuva-laakari/lahikouluttajat', value)).data
        this.kouluttajatAndVastuuhenkilot.push(kouluttaja)
        this.form.arvioinninAntaja = kouluttaja
        this.$bvModal.hide('confirm')
        toastSuccess(this, this.$t('uusi-kouluttaja-lisatty'))
      } catch (err) {
        toastFail(
          this,
          this.$t('uuden-kouluttajan-lisaaminen-epaonnistui', {
            virhe: err.response.data.title
          })
        )
      }
      params.saving = false
    }

    get account() {
      return store.getters['auth/account']
    }

    get avatar() {
      if (this.account) {
        return this.account.avatar
      }
      return undefined
    }
  }
</script>
