<template>
  <div class="fly-input fly-input--default fly-input--disabled">
    <input
      v-if="isInput"
      ref="inputRef"
      :value="computedValue"
      type="text"
      v-bind="$attrs"
      :disabled="disabled"
      v-on="listeners"
    />
  </div>
</template>

<script lang="ts">
import Vue, { defineComponent, ref, computed, watch, nextTick } from '~/.nuxt/imports';

type InputTag = "input" | "textarea";

type InputVariant = "default" | "plain";

type CallbackFunction = (...payload: any[]) => void;

export default defineComponent({
  inheritAttrs: false,
  props: {
    as: {
      type: String as () => InputTag,
      default: "input",
      validator: (value: InputTag) =>
        typeof value === "string" && ["input", "textarea"].includes(value),
    },

    variant: {
      type: String as () => InputVariant,
      default: "default",
      validator: (value: InputVariant) =>
        typeof value === "string" && ["default", "plain"].includes(value),
    },

    disabled: {
      type: Boolean,
      default: false,
    },

    value: {
      type: [String, Number],
      default: "",
    },

    minValue: {
      type: Number,
      default: undefined,
    },

    maxValue: {
      type: Number,
      default: undefined,
    },

    lazy: {
      type: Boolean,
      default: false,
    },

    decimal: {
      type: Boolean,
      default: false,
    },
  },
  emits: ["input", "focus", "blur"],
  setup(props, { emit, listeners: _listeners }) {
    const inputRef = ref<HTMLInputElement>();

    const isInput = props.as === "input";

    const focused = ref(false);

    const newValue = ref<string | number | undefined | null>(props.value);

    const computedValue = computed({
      get() {
        console.log(newValue.value);
        return newValue.value;
      },
      set(valueToSet) {
        newValue.value = valueToSet;
        emit("input", valueToSet);
      },
    });

    watch(
      () => props.value,
      (newWatchedValue) => {
        // small hack dueo to v-mask cant update itself if
        // symbols length is equal
        if (
          newValue.value?.toString().length !==
            newWatchedValue?.toString().length
        ) {
          newValue.value = "";

          nextTick(() => {
            newValue.value = newWatchedValue;
          });
        } else {
          newValue.value = newWatchedValue;
        }
      }
    );

    function updateValue(value: string) {
      if (props.decimal) {
        computedValue.value = value === "" ? null : value;

        return;
      }

      if (props.maxValue === undefined && props.minValue === undefined) {
        computedValue.value = value;
        return;
      }

      const numberValue = Number(value);

      if (
        props.minValue !== undefined &&
        (Number.isNaN(numberValue) || numberValue < props.minValue)
      ) {
        computedValue.value = Number(props.minValue);
      } else if (
        props.maxValue !== undefined &&
        (Number.isNaN(numberValue) || numberValue > props.maxValue)
      ) {
        computedValue.value = Number(props.maxValue);
      } else {
        computedValue.value = numberValue;
      }
    }

    function onInput(event: Event) {
      if (!props.lazy) {
        const target = event.target as HTMLInputElement;
        updateValue(target.value);
      }
    }

    // works only if input is lazy, duo to v-model priority functional, cuased by v-mask work
    function onChange(event: Event) {
      if (props.lazy) {
        const target = event.target as HTMLInputElement;
        updateValue(target.value);
      }
    }

    function onFocus(event: FocusEvent) {
      focused.value = true;
      emit("focus", event);
    }

    function onBlur(event: FocusEvent) {
      focused.value = false;
      emit("blur", event);
    }

    function focus() {
      inputRef.value?.focus();
    }

    const listeners = computed(() => ({
      ..._listeners,
      input: onInput,
      change: onChange,
      focus: onFocus,
      blur: onBlur,
    }));

    return {
      isInput,
      computedValue,
      focused,
      listeners,
      focus,
    };
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.fly-input {
  display: inline-flex;
  position: relative;

  align-items: center;
  max-width: 100%;
  width: 100%;
  overflow: hidden;

  background: #fff;
  border: 1px solid #A19FA9;
  border-radius: 6px;
  box-shadow: 0px 0px 0px 0px #A19FA9 inset;
  background-origin: border-box;
  transition: border-color 0.25s ease, box-shadow 0.25s ease;
}

.fly-input input {
  max-width: 100%;
  width: 100%;
  height: auto;
  padding: 10px 17px;
  background: transparent;
  color: #434054;
  font-weight: 400;
  font-size: 14px;
  line-height: 24px;
}

.fly-input input::placeholder {
  color: #A19FA9;
}

.fly-input--disabled {
  cursor: default;
  color: #D0CFD4;
  border-color: #E8E8EA;
}

.fly-input:hover:not(.fly-input--disabled):not(.fly-input--invalid),
.fly-input--focused:not(.fly-input--invalid) {
  border-color: #C48EF9;
  box-shadow: 0px 0px 0px 2px #C48EF9 inset;
}

.fly-input--invalid:not(--focused) {
  color: #FF6666;
  border-color: #FF6666;
}

.fly-input--invalid:hover:not(.fly-input--disabled),
.fly-input--invalid.fly-input--focused {
  color: #FF6666;
  border-color: #FF6666;
  box-shadow: 0px 0px 0px 2px #FF6666 inset;
}

.fly-input__sub-content {
  display: flex;
  align-items: center;
}

.fly-input__sub-content .fly-input--left {
  left: 0;
  padding-left: 16px;
}

.fly-input__sub-content .fly-input--right {
  right: 0;
  padding-right: 16px;
}

.fly-input--large input,
textarea {
  padding: 20px 24px;
}

.fly-input--rounded input,
textarea {
  border-radius: 24px;
}

.fly-input--rounded.fly-input--large input,
textarea {
  border-radius: 50px;
}
</style>
