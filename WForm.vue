<script lang="ts" setup>
import type { z } from 'zod';

import type WInput from './WInput.vue';

type WInputType = typeof WInput;

const props = defineProps<{
  schema?: z.AnyZodObject;
}>();

const emits = defineEmits<{
  (e: 'errors', errors: string[]): void;
  (e: 'submit'): void;
}>();

const zodFieldErrorMessage = (
  prop: string,
  errors: z.ZodError
): string | boolean => {
  const formatedErrors = errors.format();
  return formatedErrors[prop as string]?._errors[0] || '';
};

const wInputs = ref<WInputType[]>([]);

const registerWInput = (wInput: WInputType): void => {
  wInputs.value.push(wInput);
};

const validateWInput = (prop: string, value: string): boolean | string => {
  if (!props.schema) {
    return true;
  }

  const result = props.schema.partial().safeParse({ [prop]: value });

  return result.success || zodFieldErrorMessage(prop, result.error);
};

const onSubmit = (payload: Event): void => {
  payload.preventDefault();

  const errors: string[] = [];

  for (const wInput of wInputs.value) {
    wInput.props.error = '';

    const prop = wInput.props.name;
    const value = wInput.props.modelValue || '';

    const result = validateWInput(prop, value);

    if (typeof result === 'string') {
      wInput.exposed.setError(result);
      errors.push(`${wInput.props.label ?? wInput.props.name} - ${result}`);
    }
  }

  if (errors.length > 0) {
    emits('errors', errors);
    return;
  }

  emits('submit');
};

provide('wInputContext', () => ({
  registerWInput,
  validateWInput
}));
</script>

<template>
  <form @submit="onSubmit">
    <slot />
  </form>
</template>
