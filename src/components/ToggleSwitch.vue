<template>
  <label class="switch-wrapper">
    <span v-if="offLabel" style="margin-right: 10px;">{{ offLabel }}</span>

    <div class="switch">
      <input type="checkbox" :checked="toggleActive" @click="onToggle">
      <span class="slider"></span>
    </div>

    <span>{{ label }}</span>
  </label>
</template>

<script>
export default {
  props: {
    label: {
      type: String
    },
    offLabel: {
      type: String,
      default: null
    },
    toggleActive: {
      type: Boolean,
      default: true
    }
  },
  methods: {
    onToggle(event) {
      this.$emit("switchToggled", event.target.checked);
    }
  }
}
</script>

<style scoped>
.switch-wrapper {
  display: inline-flex;
  align-items: center;
}
/* The switch - the box around the slider */
.switch {
  position: relative;
  display: inline-block;
  width: 40px;
  height: 25px;
  margin-right: 10px;
}

/* Hide default HTML checkbox */
.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

/* The slider */
.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  -webkit-transition: .4s;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 15px;
  width: 15px;
  left: 6px;
  bottom: 5px;
  background-color: white;
  -webkit-transition: .4s;
  transition: .4s;
}

input:checked + .slider {
  background-color: var(--color-black);
}

input:focus + .slider {
  box-shadow: 0 0 1px var(--color-black);
}

input:checked + .slider:before {
  -webkit-transform: translateX(13px);
  -ms-transform: translateX(13px);
  transform: translateX(13px);
}

.slider {
  border-radius: 34px;
}

.slider:before {
  border-radius: 50%;
}
</style>