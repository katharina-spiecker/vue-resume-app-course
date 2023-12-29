<template>
  <CustomButton btn-type="primary" @click="exportPdf" style="margin-bottom: 10px;">
    Export resume as pdf
  </CustomButton>
</template>

<script>
import CustomButton from './CustomButton.vue';

export default {
  components: {
    CustomButton
  },
  props: {
    resumeFormat: {
      type: String
    }
  },
  methods: {
    exportPdf() {
      const unit = this.resumeFormat == 'a4' ? 'mm' : 'in';

      const pdfConfig = {
        margin: 0,
        filename: 'resume',
        jsPDF: {
          unit: unit,
          format: this.resumeFormat,
          orientation: 'portrait'
        }
      }
      
      const resume = document.getElementById('resume');
      html2pdf().set(pdfConfig).from(resume).save();
    }
  }
}
</script>