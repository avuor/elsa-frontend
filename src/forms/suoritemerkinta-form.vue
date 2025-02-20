<template>
  <b-form @submit.stop.prevent="onSubmit">
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
    <elsa-form-group :label="$t('oppimistavoite')" :required="true">
      <template v-slot="{ uid }">
        <elsa-form-multiselect
          :id="uid"
          v-model="form.oppimistavoite"
          :options="oppimistavoitteenKategoriat"
          :state="validateState('oppimistavoite')"
          group-values="oppimistavoitteet"
          group-label="nimi"
          :group-select="false"
          label="nimi"
          track-by="id"
        >
          <template slot="option" slot-scope="props">
            <span v-if="props.option.$isLabel">{{ props.option.$groupLabel }}</span>
            <span v-else class="ml-3">{{ props.option.nimi }}</span>
          </template>
        </elsa-form-multiselect>
        <b-form-invalid-feedback :id="`${uid}-feedback`">
          {{ $t('pakollinen-tieto') }}
        </b-form-invalid-feedback>
      </template>
    </elsa-form-group>
    <elsa-form-group :label="$t('vaativuustaso')" :required="true">
      <template #label-help>
        <elsa-popover>
          <template>
            <h3>{{ $t('vaativuustaso') }}</h3>
            <div v-for="(taso, index) in vaativuustasot" :key="index">
              <h4>{{ taso.arvo }} {{ $t(taso.nimi) }}</h4>
              <p>{{ $t(taso.kuvaus) }}</p>
            </div>
          </template>
        </elsa-popover>
      </template>
      <template v-slot="{ uid }">
        <elsa-form-multiselect
          :id="uid"
          v-model="form.vaativuustaso"
          :options="vaativuustasot"
          :state="validateState('vaativuustaso')"
          :custom-label="(value) => `${value.arvo} ${value.nimi}`"
          track-by="arvo"
        >
          <template slot="singleLabel" slot-scope="{ option }">
            <span class="font-weight-700">{{ option.arvo }}</span>
            {{ $t(option.nimi) }}
          </template>
          <template slot="option" slot-scope="{ option }">
            <span class="font-weight-700">{{ option.arvo }}</span>
            {{ $t(option.nimi) }}
          </template>
        </elsa-form-multiselect>
        <b-form-invalid-feedback :id="`${uid}-feedback`">
          {{ $t('pakollinen-tieto') }}
        </b-form-invalid-feedback>
      </template>
    </elsa-form-group>
    <b-form-row>
      <elsa-form-group
        v-if="form.oppimistavoite"
        :label="arviointiAsteikonNimi"
        :required="true"
        class="col-md-8"
      >
        <template #label-help>
          <elsa-popover>
            <template>
              <h3>{{ arviointiAsteikonNimi }}</h3>
              <div v-for="(asteikonTaso, index) in arviointiasteikko.tasot" :key="index">
                <h4>
                  {{ asteikonTaso.taso }} {{ $t('arviointiasteikon-taso-' + asteikonTaso.nimi) }}
                </h4>
                <p>{{ $t('arviointiasteikon-tason-kuvaus-' + asteikonTaso.nimi) }}</p>
              </div>
            </template>
          </elsa-popover>
        </template>
        <template v-slot="{ uid }">
          <elsa-form-multiselect
            :id="uid"
            v-model="form.arviointiasteikonTaso"
            :options="arviointiasteikko.tasot"
            :state="validateState('arviointiasteikonTaso')"
            :custom-label="(value) => `${value.taso} ${value.nimi}`"
            track-by="taso"
          >
            <template slot="singleLabel" slot-scope="{ option }">
              <span class="font-weight-700">{{ option.taso }}</span>
              {{ $t('arviointiasteikon-taso-' + option.nimi) }}
            </template>
            <template slot="option" slot-scope="{ option }">
              <span class="font-weight-700">{{ option.taso }}</span>
              {{ $t('arviointiasteikon-taso-' + option.nimi) }}
            </template>
          </elsa-form-multiselect>
          <b-form-invalid-feedback :id="`${uid}-feedback`">
            {{ $t('pakollinen-tieto') }}
          </b-form-invalid-feedback>
        </template>
      </elsa-form-group>
      <elsa-form-group :label="$t('suorituspaiva')" class="col-md-4" :required="true">
        <template v-slot="{ uid }">
          <elsa-form-datepicker
            :id="uid"
            v-model="form.suorituspaiva"
            :state="validateState('suorituspaiva')"
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
    <div class="text-right">
      <elsa-button variant="back" :to="{ name: 'suoritemerkinnat' }">
        {{ $t('peruuta') }}
      </elsa-button>
      <elsa-button
        v-if="value.id"
        :loading="params.deleting"
        variant="outline-danger"
        @click="onSuoritemerkintaDelete"
      >
        {{ $t('poista-merkinta') }}
      </elsa-button>
      <elsa-button :loading="params.saving" type="submit" variant="primary" class="ml-2">
        {{ $t('tallenna') }}
      </elsa-button>
    </div>
  </b-form>
</template>

<script lang="ts">
  import Component from 'vue-class-component'
  import { Mixins, Prop } from 'vue-property-decorator'
  import { validationMixin } from 'vuelidate'
  import { required } from 'vuelidate/lib/validators'

  import ElsaButton from '@/components/button/button.vue'
  import ElsaFormDatepicker from '@/components/datepicker/datepicker.vue'
  import ElsaFormGroup from '@/components/form-group/form-group.vue'
  import ElsaFormMultiselect from '@/components/multiselect/multiselect.vue'
  import ElsaPopover from '@/components/popover/popover.vue'
  import TyoskentelyjaksoForm from '@/forms/tyoskentelyjakso-form.vue'
  import TyoskentelyjaksoMixin from '@/mixins/tyoskentelyjakso'
  import {
    ArviointiasteikonTaso,
    Erikoisala,
    Kunta,
    OppimistavoitteenKategoria,
    Suoritemerkinta,
    Vaativuustaso
  } from '@/types'
  import { vaativuustasot, ArviointiasteikkoTyyppi } from '@/utils/constants'

  @Component({
    components: {
      TyoskentelyjaksoForm,
      ElsaFormGroup,
      ElsaFormMultiselect,
      ElsaPopover,
      ElsaFormDatepicker,
      ElsaButton
    },
    validations: {
      form: {
        tyoskentelyjakso: {
          required
        },
        oppimistavoite: {
          required
        },
        vaativuustaso: {
          required
        },
        arviointiasteikonTaso: {
          required
        },
        suorituspaiva: {
          required
        }
      }
    }
  })
  export default class SuoritemerkintaForm extends Mixins(validationMixin, TyoskentelyjaksoMixin) {
    @Prop({ required: false, default: () => [] })
    oppimistavoitteenKategoriat!: OppimistavoitteenKategoria[]

    @Prop({ required: false, default: () => [] })
    kunnat!: Kunta[]

    @Prop({ required: false, default: () => [] })
    erikoisalat!: Erikoisala[]

    @Prop({
      required: false,
      default: () => ({
        tyoskentelyjakso: null,
        oppimistavoite: null,
        vaativuustaso: null,
        arviointiasteikonTaso: null,
        suorituspaiva: null,
        lisatiedot: null
      })
    })
    value!: Suoritemerkinta

    form?: Suoritemerkinta
    vaativuustasot = vaativuustasot
    params = {
      saving: false,
      deleting: false
    }

    mounted() {
      this.form = {
        tyoskentelyjakso: this.value.tyoskentelyjakso,
        oppimistavoite: this.value.oppimistavoite,
        vaativuustaso: vaativuustasot.find((taso) => taso.arvo === this.value.vaativuustaso),
        arviointiasteikonTaso: this.oppimistavoitteenKategoriat
          .find((kategoria) => kategoria.id === this.value.oppimistavoite?.kategoriaId)
          ?.arviointiasteikko.tasot.find(
            (asteikonTaso) => asteikonTaso.taso === this.value.arviointiasteikonTaso
          ),
        suorituspaiva: this.value.suorituspaiva,
        lisatiedot: this.value.lisatiedot
      }
    }

    validateState(name: string) {
      const { $dirty, $error } = this.$v.form[name] as any
      return $dirty ? ($error ? false : null) : null
    }

    get arviointiasteikko() {
      return this.oppimistavoitteenKategoriat.find(
        (kategoria) => kategoria.id === this.form?.oppimistavoite.kategoriaId
      )?.arviointiasteikko
    }

    get arviointiAsteikonNimi() {
      return this.arviointiasteikko?.nimi === ArviointiasteikkoTyyppi.EPA
        ? this.$t('luottamuksen-taso')
        : this.$t('etappi')
    }

    onSubmit() {
      this.$v.form.$touch()
      if (this.$v.form.$anyError) {
        return
      }
      this.$emit(
        'submit',
        {
          tyoskentelyjaksoId: this.form?.tyoskentelyjakso.id,
          oppimistavoiteId: this.form?.oppimistavoite.id,
          vaativuustaso: (this.form?.vaativuustaso as Vaativuustaso).arvo,
          arviointiasteikonTaso: (this.form?.arviointiasteikonTaso as ArviointiasteikonTaso).taso,
          suorituspaiva: this.form?.suorituspaiva,
          lisatiedot: this.form?.lisatiedot
        },
        this.params
      )
    }

    onSuoritemerkintaDelete() {
      this.$emit('delete', this.params)
    }
  }
</script>
