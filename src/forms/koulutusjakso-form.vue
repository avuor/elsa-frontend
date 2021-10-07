<template>
  <b-form @submit.stop.prevent="onSubmit">
    <elsa-form-group :label="$t('koulutusjakson-nimi')" :required="true">
      <template v-slot="{ uid }">
        <b-form-input :id="uid" v-model="form.nimi" :state="validateState('nimi')"></b-form-input>
        <b-form-invalid-feedback :id="`${uid}-feedback`">
          {{ $t('pakollinen-tieto') }}
        </b-form-invalid-feedback>
      </template>
    </elsa-form-group>
    <elsa-form-group
      :label="$t('tyoskentelyjakso')"
      :add-new-enabled="true"
      :add-new-label="$t('lisaa-tyoskentelyjakso')"
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
        <div
          :id="uid"
          v-for="(tyoskentelyjakso, index) in form.tyoskentelyjaksot"
          :key="index"
          class="mb-2"
        >
          <elsa-form-multiselect
            :id="uid"
            v-model="form.tyoskentelyjaksot[index]"
            :options="tyoskentelyjaksotFormatted"
            label="label"
            track-by="id"
            @select="onTyoskentelyjaksoSelect"
          />
          <elsa-button
            v-if="index !== 0"
            @click="deleteTyoskentelyjakso(index)"
            variant="link"
            size="sm"
            class="text-decoration-none shadow-none p-0"
          >
            <font-awesome-icon :icon="['far', 'trash-alt']" fixed-width size="sm" />
            {{ $t('poista-tyoskentelyjakso') }}
          </elsa-button>
        </div>
        <elsa-button
          @click="addTyoskentelyjakso"
          variant="link"
          size="sm"
          class="text-decoration-none shadow-none p-0"
        >
          <font-awesome-icon icon="plus" fixed-width size="sm" />
          {{ $t('useampi-jakso') | lowercase }}
        </elsa-button>
      </template>
    </elsa-form-group>
    <elsa-form-group :label="$t('osaamistavoitteet-omalta-erikoisalalta')">
      <template v-slot="{ uid }">
        <elsa-form-multiselect
          :id="uid"
          v-model="form.osaamistavoitteet"
          :options="osaamistavoitteetFormatted"
          label="label"
          track-by="id"
          :allow-empty="true"
          :multiple="true"
          :taggable="true"
          @tag="onOsaamistavoiteTag"
        />
      </template>
    </elsa-form-group>
    <elsa-form-group :label="$t('muut-osaamistavoitteet')">
      <template v-slot="{ uid }">
        <b-form-textarea :id="uid" v-model="form.muutOsaamistavoitteet" rows="5" />
      </template>
    </elsa-form-group>
    <hr />
    <div class="d-flex flex-row-reverse flex-wrap">
      <elsa-button :loading="params.saving" type="submit" variant="primary" class="ml-2 mb-2">
        {{ $t('tallenna-koulutusjakso') }}
      </elsa-button>
      <elsa-button variant="back" @click.stop.prevent="onCancel" class="mb-2">
        {{ $t('peruuta') }}
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
  import ElsaFormGroup from '@/components/form-group/form-group.vue'
  import ElsaFormMultiselect from '@/components/multiselect/multiselect.vue'
  import TyoskentelyjaksoForm from '@/forms/tyoskentelyjakso-form.vue'
  import TyoskentelyjaksoMixin from '@/mixins/tyoskentelyjakso'
  import { Erikoisala, Koulutusjakso, Kunta } from '@/types'

  @Component({
    components: {
      ElsaFormGroup,
      ElsaButton,
      ElsaFormMultiselect,
      TyoskentelyjaksoForm
    },
    validations: {
      form: {
        nimi: {
          required
        }
      }
    }
  })
  export default class KoulutusjaksoForm extends Mixins(validationMixin, TyoskentelyjaksoMixin) {
    @Prop({ required: false, default: () => [] })
    kunnat!: Kunta[]

    @Prop({ required: false, default: () => [] })
    erikoisalat!: Erikoisala[]

    @Prop({
      required: false,
      type: Object,
      default: () => ({
        id: null,
        nimi: null,
        muutOsaamistavoitteet: null,
        luotu: null,
        tallennettu: null,
        lukittu: false,
        tyoskentelyjaksot: [
          {
            alkamispaiva: null,
            paattymispaiva: null,
            minPaattymispaiva: null,
            osaaikaprosentti: 100,
            tyoskentelypaikka: {
              nimi: null,
              kunta: { abbreviation: null },
              tyyppi: null,
              muuTyyppi: null
            },
            kaytannonKoulutus: null,
            omaaErikoisalaaTukeva: null,
            hyvaksyttyAiempaanErikoisalaan: null
          }
        ],
        osaamistavoitteet: [],
        koulutussuunnitelma: null
      })
    })
    value!: Koulutusjakso
    form!: Koulutusjakso
    params = {
      saving: false
    }
    osaamistavoitteetFormatted = [
      {
        id: 1,
        label: 'Neuvonta ja ohjaus'
      },
      {
        id: 2,
        label: 'Työterveystoiminnan suunnittelu'
      },
      {
        id: 3,
        label: 'Lorem ipsum'
      }
    ]

    mounted() {
      this.form = this.value
    }

    validateState(name: string) {
      const { $dirty, $error } = this.$v.form[name] as any
      return $dirty ? ($error ? false : null) : null
    }

    onOsaamistavoiteTag(value: { id: number; label: string }) {
      console.log(value)
    }

    addTyoskentelyjakso() {
      this.form.tyoskentelyjaksot.push({
        alkamispaiva: null,
        paattymispaiva: null,
        minPaattymispaiva: null,
        osaaikaprosentti: 100,
        tyoskentelypaikka: {
          nimi: null,
          kunta: { abbreviation: null },
          tyyppi: null,
          muuTyyppi: null
        },
        kaytannonKoulutus: null,
        omaaErikoisalaaTukeva: null,
        hyvaksyttyAiempaanErikoisalaan: null
      })
    }

    deleteTyoskentelyjakso(index: number) {
      const tyoskentelyjaksot = [...this.form.tyoskentelyjaksot]
      tyoskentelyjaksot.splice(index, 1)
      this.form = {
        ...this.form,
        tyoskentelyjaksot
      }
    }

    onSubmit() {
      this.$v.form.$touch()
      if (this.$v.form.$anyError) {
        return
      }
      this.$emit('submit', this.form, this.params)
    }

    onCancel() {
      this.$emit('cancel')
    }
  }
</script>

<style lang="scss" scoped>
  @import '~@/styles/variables';
  @import '~bootstrap/scss/mixins/breakpoints';
</style>
