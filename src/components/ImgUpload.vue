<template>
  <label>
    Upload Image
    <input
      type="file"
      accept="image/jpeg image/png image/jpg"
      @change="changeImage" />
  </label>
</template>

<script>
export default {
  methods: {
    changeImage(event) {
      const selectedFile = event.target.files[0];
      if (selectedFile) {
        const reader = new FileReader();

        reader.addEventListener("load", () => {
          this.$emit("imageChanged", reader.result);
        });

        reader.readAsDataURL(selectedFile);
      }
    },
  },
};
</script>

<style scoped>
/* hide browser rendered input button. We will render our own button */
input[type="file"] {
  display: none;
}

label {
  display: block;
  border-radius: 4px;
  padding: 8px 10px;
  text-align: center;
  border: none;
  font-size: 16px;
  background-color: #82c0cc;
}
</style>
