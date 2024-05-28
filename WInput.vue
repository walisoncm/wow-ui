<script lang="ts" setup>
import type { ComponentInternalInstance, InputTypeHTMLAttribute } from 'vue';

interface WInputContextFunctions {
  registerWInput: (wInput: ComponentInternalInstance) => void;
  validateWInput: (prop: string, value: string) => boolean | string;
}

type WInputContext = () => WInputContextFunctions;

const wInputContext = inject<WInputContext>('wInputContext', () => ({
  registerWInput: (_wInput: ComponentInternalInstance) => {},
  validateWInput: (_prop: string, _value: string) => true
}));

const { registerWInput, validateWInput } = wInputContext();
const instance = getCurrentInstance();

if (instance) {
  registerWInput(instance);
}

const props = defineProps<{
  modelValue?: string;
  type?: InputTypeHTMLAttribute;
  icon?: string;
  label?: string;
  placeholder?: string;
  hint?: string;
  clearable?: boolean;
  rules?: ((value: string) => boolean | string)[]
  name?: string;
  error?: string;
}>();

const emits = defineEmits<{
  (e: 'update:modelValue', value: string): void;
}>();

const id = useId();

interface State {
  value: string;
  error: string;
}

const state = reactive<State>({
  value: props.modelValue ?? '',
  error: ''
});

const onClear = (): void => {
  state.value = '';
  emits('update:modelValue', '');
};

const setError = (error: string): void => {
  state.error = error;
};

const checkError = (result: boolean | string): boolean => {
  if (typeof result === 'string') {
    setError(result);
    return true;
  }

  setError('');
  return false;
};

const onUpdateModelValue = (value: string): void => {
  emits('update:modelValue', value);

  if (props.name) {
    if (checkError(validateWInput(props.name, value))) {
      return;
    }
  }

  if (props.rules) {
    for (const rule of props.rules) {
      if (checkError(rule(value))) {
        return;
      }
    }
  }
};

watch(
  () => props.modelValue,
  (value?: string) => state.value = value ?? ''
);

defineExpose({
  setError
});
</script>

<template>
  <div class="w-input mb-6">
    <div
      class="
        w-input__container
        relative
        flex
        border
        rounded
        px-2
        min-h-14
        border-gray-400
        hover:border-gray-500
      "
      :class="{
        'w-input--not-empty': !!state.value,
        'w-input--error': !!state.error || !!props.error,
        'w-input--with-icon': !!props.icon,
        'w-input--with-label': !!props.label
      }"
    >
      <div
        v-if="!!props.icon"
        class="
          w-input__prepend
          flex
          items-center
          text-2xl
          pr-2
          text-gray-400
        "
      >
        <span :class="props.icon" />
      </div>

      <input
        :id="id"
        v-model="state.value"
        class="
          block
          outline-none
          appearence-none
          w-full
          leading-tight
          py-2
          bg-transparent
        "
        :type="props.type ?? 'text'"
        :placeholder="props.placeholder"
        @update:model-value="onUpdateModelValue"
      >

      <label
        v-if="!!props.label"
        class="
          absolute
          top-0
          left-2
          block
          cursor-text
          select-none
          leading-[3.5rem]
          transition-all
          duration-300
          text-gray-400
        "
        :for="id"
      >
        {{ props.label }}
      </label>

      <div
        v-if="!!state.error || !!props.error"
        class="
          w-input__append
          flex
          items-center
          text-2xl
          pl-2
          text-red-500
        "
      >
        <span class="i-ic-round-error" />
      </div>

      <div
        v-if="!!props.clearable"
        class="
          w-input__clear
          flex
          items-center
          text-2xl
          pl-2
          text-gray-400
        "
      >
        <span
          class="
            i-ic-round-cancel
            cursor-pointer
          "
          @click="onClear()"
        />
      </div>
    </div>
    <p
      v-if="!!state.error || !!props.error || !!props.hint"
      class="
        py-1
        px-2
        text-xs
        text-gray-400
      "
    >
      {{ state.error || props.error || props.hint }}
    </p>
  </div>
</template>

<style lang="scss" scoped>
.w-input__container {

  &.w-input--with-icon {
    &>label {
      left: 2.5rem;
    }

    &.w-input--not-empty>label,
    &:focus-within>label,
    &>input:placeholder-shown+label {
      left: 2.5rem;
    }
  }

  &.w-input--with-label>input {
    padding-top: 1.5rem;
  }

  &.w-input--error {
    border-color: red;

    &>.w-input__prepend,
    &>label,
    &+p {
      color: red;
    }
  }

  &.w-input--not-empty>label,
  &:focus-within>label,
  &>input:placeholder-shown+label {
    left: 0.5rem;
    line-height: 1.5rem;
    font-size: 0.75rem;
  }

  &:focus-within:not(&.w-input--error) {
    border-color: var(--company-color-1);

    &>.w-input__prepend,
    &>label {
      color: var(--company-color-1);
    }
  }
}
</style>
