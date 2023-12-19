<template>
  <label>
    <input
      type="number"
      class="custom-input"
      :min="min"
      :max="max"
      :value="currentValue"
      @change="$emit('percentageChanged', $event.target.value)" />
    % {{ label }}
  </label>
</template>

<script>
export default {
  props: {
    label: {
      type: String,
    },
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100,
    },
    currentValue: {
      type: Number,
      default: 0,
    },
  },
};
</script>

<style scoped>
label {
  display: flex;
  align-items: center;
}

input {
  width: 50px;
  margin-right: 8px;
  display: block;
}
</style>
