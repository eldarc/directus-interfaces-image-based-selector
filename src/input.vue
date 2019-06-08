<template>
  <div
    :id="name"
    class="selector"
  >
    <div
      v-for="(choice, val) in choices"
      :key="val"
      :class="['option', {selected: value && value.includes(val)}, {readonly: readonly}]"
      @click="updateValue(val)"
    >
      <img
        :src="choice.image"
        :alt="choice.label"
        class="image"
      >

      <div class="label">
        {{ choice.label }}
      </div>
    </div>
  </div>
</template>

<script>
import mixin from '@directus/extension-toolkit/mixins/interface'

export default {
  mixins: [mixin],
  computed: {
    choices () {
      let choices = this.options.choices

      if (!choices) return {}

      if (typeof this.options.choices === 'string') {
        choices = JSON.parse(this.options.choices)
      }

      return choices
    }
  },
  created () {
    // Check if loaded values exist in choices.
    let verifiedValues = this.value.filter(item => Object.keys(this.options.choices).find(key => key === item))

    if (verifiedValues.length > 0 && this.options.wrapWithDelimiter) {
      verifiedValues = [
        '',
        ...verifiedValues,
        ''
      ]
    }

    this.$emit('input', verifiedValues)
  },
  methods: {
    updateValue (targetValue) {
      if (this.readonly) {
        return
      }

      let newValue = []
      if (this.options.multiselect) {
        let valueArray

        // Convert string to an array or remove the first and last item if `wrapWithDelimiter` option is enabled.
        if (typeof this.value === 'string') {
          valueArray = [targetValue]
        } else {
          valueArray = this.value.length > 1 && this.value[0] === '' && this.value[this.value.length - 1] === '' ? this.value.slice(1, -1) : this.value
        }

        // Check if the item is already selected.
        const isSelected = valueArray.findIndex(item => item === targetValue)

        // If selected remove from the collection, if not selected add to the selection.
        if (isSelected !== -1) {
          newValue = valueArray.filter(item => item !== targetValue)
        } else {
          newValue = [
            ...valueArray,
            targetValue
          ]
        }
      } else {
        // In single mode select only one value.
        newValue = [targetValue]
      }

      // Append and prepend items when `wrapWithDelimiter` option is enabled.
      if (newValue.length > 0 && this.options.wrapWithDelimiter) {
        newValue = [
          '',
          ...newValue,
          ''
        ]
      }

      this.$emit('input', newValue)
    }
  }
}
</script>

<style lang="scss" scoped>
.selector {
  .option {
    display: inline-table;
    margin-right: 1rem;
    margin-bottom: 1rem;
    opacity: 0.7;
    cursor: pointer;
    transition: all var(--fast) var(--transition);
    text-align: center;

    &.readonly {
      cursor: not-allowed;
    }

    &:last-of-type {
      margin-right: 0;
    }

    .image {
      transition: all var(--fast) var(--transition);
      border: var(--input-border-width) solid var(--lighter-gray);
      border-radius: var(--border-radius);
    }

    &:hover:not(.readonly) {
      opacity: 1;

      .image {
        border-color: var(--light-gray);
      }
    }

    &.selected, &:focus {
      transform: scale(1.1);
      opacity: 1;

      .image {
        border-color: var(--darker-gray);
      }
    }

    .label {
      display: table-caption;
      caption-side: bottom;
    }
  }
}
</style>
