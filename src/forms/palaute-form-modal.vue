<template>
  <b-modal
    id="palauteFormModal"
    centered
    :title="$t('palaute-otsikko')"
    @hide="hideAndReset"
    size="lg"
    :hide-footer="true"
  >
    <div v-if="showFormSent">
      <div class="text-center mb-5">
        <font-awesome-icon :icon="['fas', 'check-circle']" class="mt-5 mb-3 text-success icon-xl" />
        <p class="font-weight-500 text-center">{{ $t('palautteen-lahettaminen-onnistui') }}</p>
      </div>
      <div class="text-right mr-2">
        <elsa-button variant="primary" class="px-5 mb-2" :loading="saving" @click="hideAndReset">
          {{ $t('sulje') }}
        </elsa-button>
      </div>
    </div>
    <div v-else-if="showError">
      <div class="text-center mb-5">
        <font-awesome-icon
          :icon="['fas', 'exclamation-circle']"
          class="mt-5 mb-3 text-danger icon-xl"
        />
        <p class="font-weight-500 text-center">
          {{ $t('palautteen-lahettaminen-epaonnistui') }}
        </p>
      </div>
      <div class="text-right mr-2">
        <elsa-button variant="primary" class="px-5 mb-2" :loading="saving" @click="hideAndReset">
          {{ $t('sulje') }}
        </elsa-button>
      </div>
    </div>
    <div v-else>
      <p>{{ $t('palaute-ingressi') }}</p>
      <elsa-form-group :label="$t('palautteen-aihe')" :required="true">
        <template v-slot="{ uid }">
          <b-form-radio
            v-model="form.palautteenAihe"
            name="palautteen-aihe"
            :state="validateState('palautteenAihe')"
            :value="$t('palautteen-aihe-kehitysidea')"
          >
            {{ $t('palautteen-aihe-kehitysidea') }}
          </b-form-radio>
          <b-form-radio
            v-model="form.palautteenAihe"
            name="palautteen-aihe"
            :state="validateState('palautteenAihe')"
            :value="$t('palautteen-aihe-virhe')"
          >
            {{ $t('palautteen-aihe-virhe') }}
          </b-form-radio>
          <b-form-radio
            v-model="form.palautteenAihe"
            name="palautteen-aihe"
            :state="validateState('palautteenAihe')"
            :value="$t('palautteen-aihe-muu')"
          >
            {{ $t('palautteen-aihe-muu') }}
          </b-form-radio>
          <b-form-invalid-feedback :id="`${uid}-feedback`" :state="validateState('palautteenAihe')">
            {{ $t('pakollinen-tieto') }}
          </b-form-invalid-feedback>
        </template>
      </elsa-form-group>
      <elsa-form-group :label="$t('palaute')" :required="true">
        <template v-slot="{ uid }">
          <b-form-textarea
            :id="uid"
            v-model="form.palaute"
            :state="validateState('palaute')"
            rows="7"
            class="textarea-min-height"
          ></b-form-textarea>
          <b-form-invalid-feedback :id="`${uid}-feedback`">
            {{ $t('pakollinen-tieto') }}
          </b-form-invalid-feedback>
        </template>
      </elsa-form-group>
      <div class="text-right mt-4">
        <elsa-button variant="back" :disabled="saving" @click="hideAndReset">
          {{ $t('peruuta') }}
        </elsa-button>
        <elsa-button variant="primary" class="ml-4 px-5" :loading="saving" @click="onSubmit">
          {{ $t('palaute-laheta') }}
        </elsa-button>
      </div>
    </div>
  </b-modal>
</template>

<script lang="ts">
  import axios from 'axios'
  import Component from 'vue-class-component'
  import { Mixins, Prop, Watch } from 'vue-property-decorator'
  import { validationMixin } from 'vuelidate'
  import { required } from 'vuelidate/lib/validators'

  import ElsaButton from '@/components/button/button.vue'
  import ElsaFormGroup from '@/components/form-group/form-group.vue'
  import { Palaute } from '@/types'

  @Component({
    components: {
      ElsaFormGroup,
      ElsaButton
    },
    validations: {
      form: {
        palautteenAihe: {
          required
        },
        palaute: {
          required
        }
      }
    }
  })
  export default class PalauteFormModal extends Mixins(validationMixin) {
    @Prop({ required: true, type: Boolean, default: false })
    show!: boolean

    saving = false
    showFormSent = false
    showError = false

    form: Palaute = {
      palautteenAihe: null,
      palaute: null
    }

    validateState(name: string) {
      const { $dirty, $error } = this.$v.form[name] as any
      return $dirty ? ($error ? false : null) : null
    }

    @Watch('show')
    onPropertyChanged(value: boolean) {
      if (value) {
        this.$bvModal.show('palauteFormModal')
      } else {
        this.$bvModal.hide('palauteFormModal')
      }
    }

    hideAndReset() {
      this.$emit('hide')
      this.$v.form.$reset()
      this.form = {
        palautteenAihe: null,
        palaute: null
      }
      this.showFormSent = false
      this.showError = false
    }

    async onSubmit() {
      this.$v.form.$touch()
      if (this.$v.form.$anyError) {
        return
      }

      this.saving = true
      try {
        await axios.post('/palaute', this.form)
        this.showFormSent = true
      } catch {
        this.showError = true
      }
      this.saving = false
    }
  }
</script>
