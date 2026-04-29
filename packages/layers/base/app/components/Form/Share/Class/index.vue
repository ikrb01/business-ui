<script setup lang="ts">
import type { Form, FormErrorEvent } from '@nuxt/ui'

const props = defineProps<{
  variant: 'add' | 'edit'
  name?: string
  title: string
  stateKey: string
  nested?: boolean
  validationContext?: { existingNames: string[] }
}>()

const emit = defineEmits<{
  done: []
  cancel: []
  remove: []
}>()

const { t } = useI18n()
const { alerts, attachAlerts } = useFilingAlerts(props.stateKey)

const formTarget = 'share-class-form'

const currencyOptions = getCurrencyList().map(c => ({
  code: c.code,
  label: `${c.name}, ${c.code}`
}))

const schema = computed(() =>
  getActiveShareClassSchema(props.validationContext)
)
const labelId = useId()

const model = defineModel<ShareClassSchema>({ required: true })
const formRef = useTemplateRef<Form<ShareClassSchema>>('share-class-form')

const formErrors = computed(() => {
  const errors = formRef.value?.getErrors()

  return {
    name: !!errors?.find(e => e.name?.includes('name')),
    maxShares: !!errors?.find(e => e.name?.includes('maxNumberOfShares')),
    parValue: !!errors?.find(e => e.name?.includes('parValue'))
  }
})

function resetFields(fields: 'parValue' | 'maxShares') {
  if (fields === 'parValue') {
    model.value.parValue = null
    model.value.currency = undefined
    formRef.value?.clear(/^(parValue|currency)$/)
  } else {
    model.value.maxNumberOfShares = null
    formRef.value?.clear(/^maxNumberOfShares$/)
  }
}

async function onDone() {
  try {
    await formRef.value?.validate()
    emit('done')
  } catch (e) {
    onFormSubmitError(e as FormErrorEvent)
  }
}

const { targetId, messageId } = attachAlerts(formTarget, model)

const nameInputSlots = computed(() => ({
  trailing: h(
    'span',
    { class: ['text-base font-bold', formErrors.value.name ? 'text-error' : ''] },
    t('label.shares')
  )
}))

provide('UInput-slots-share-class-name-input', nameInputSlots)
provide('UInput-props-max-number-shares-input', { maxlength: '17' })
provide('UInput-props-par-value-input', { maxlength: '17' })

/**
 * ✅ SINGLE, CORRECT onMounted
 */
onMounted(async () => {
  await nextTick()
  formRef.value?.clear()

  if (model.value.currencyAdditional) {
    model.value.currency = ''

    await nextTick()

    await formRef.value?.validate({
      name: 'currency',
      silent: true
    })
  }
})

defineExpose({
  formRef
})
</script>

<template>
  <UForm
    ref="share-class-form"
    :data-testid="`${variant}-share-class-form`"
    :schema="schema"
    :name
    :nested
    :state="model"
    @keydown.enter.prevent.stop="onDone"
  >
    <!-- template unchanged -->
  </UForm>
</template>