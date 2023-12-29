<template>
  <main class="container">
    <Sidebar>
      <ToggleSwitch
        @switch-toggled="toggleEditMode"
        label="Edit mode"
        off-label="Export Mode"
        :toggle-active="editing"
      />

      <div class="sidebar-section" v-if="!editing">
        <SelectInput
          label="Resume format"
          :options="[{'name': 'A4', 'value': 'a4'}, {'name': 'Letter', 'value': 'letter'}]"
          :default-option="resumeFormat"
          @update-selection="resumeFormat = $event"
        />
        <ExportPdf :resume-format="resumeFormat"/>
      </div>

      <div class="sidebar-section" v-if="editing">
        <div class="sidebar-title">Left column</div>
        <ColorInput
          label="Highlight color"
          :default-color="colors.left.highlight"
          @color-changed="colors.left.highlight = $event" />

        <ColorInput
          label="Background color"
          :default-color="colors.left.background"
          @color-changed="colors.left.background = $event" />

        <ColorInput
          label="Text color"
          :default-color="colors.left.text"
          @color-changed="colors.left.text = $event" />
      </div>

      <div class="sidebar-section" v-if="editing">
        <div class="sidebar-title">Right column</div>
        <ColorInput
          label="Highlight color"
          :default-color="colors.right.highlight"
          @color-changed="colors.right.highlight = $event"
        />

        <ColorInput
          label="Background color"
          :default-color="colors.right.background"
          @color-changed="colors.right.background = $event"
        />

        <ColorInput
          label="Text color"
          :default-color="colors.right.text"
          @color-changed="colors.right.text = $event"
        />
      </div>

      <div class="sidebar-section" v-if="editing">
        <PercentageInput
          label="Width of left column"
          :min="20"
          :max="80"
          :current-value="widthLeft"
          @percentage-changed="widthLeft = $event" />
          
        <SelectInput
          label="Headline thickness"
          :options="[
            { name: 'Thin', value: '300' },
            { name: 'Medium', value: '400' },
            { name: 'Thick', value: '600' },
          ]"
          :default-option="headlineWeight"
          @update-selection="headlineWeight = $event" />
      </div>

      <div class="sidebar-section" v-if="editing">
        <ToggleSwitch
          @switch-toggled="toggleImageDisplay"
          label="Show photo"
          :toggle-active="showImage"
        />

        <SelectInput
          v-if="showImage"
          label="Photo shape"
          :options="[
            { name: 'Square', value: 'square' },
            { name: 'Round', value: 'round' },
          ]"
          :default-option="imageShape"
          @update-selection="imageShape = $event"
        />

        <ImgUpload
          v-if="showImage"
          @image-changed="imageUrl = $event"
        />
      </div>
      

    </Sidebar>

    <div class="resume-wrapper">
      <CustomButton btn-type="primary-right" @click="saveConfig">Save configuration in browser</CustomButton>
      <div
        id="resume"
        class="d-flex"
        :class="{ 'edit-off': !editing, 'letter-format': resumeFormat == 'letter' }"
        :style="cssVariables">
        <div class="left-col" :style="{ width: percentageWidthLeft }">
          <ResumeSection>
            <img
              v-if="showImage"
              :src="imageUrl"
              class="profile-pic"
              :class="{ circle: imageShape == 'round' }"
              alt="profile picture" />

            <SectionHeadline
              :headline="headlines[0]"
              @headline-edited="updateHeadline($event, 0)"
              :editing="editing" />

            <div
              :contenteditable="editing"
              @input="updateProperty($event, 'introText')">
              {{ introText }}
            </div>
          </ResumeSection>

          <ResumeSection>
            <SectionHeadline
              :headline="headlines[1]"
              @headline-edited="updateHeadline($event, 1)"
              :editing="editing" />

            <Contact
              :contact="contact"
              @edit="updateNestedProperty"
              :editing="editing"
              :icon-color="colors.left.highlight" />
          </ResumeSection>

          <ResumeSection>
            <SectionHeadline
              :headline="headlines[2]"
              @headline-edited="updateHeadline($event, 2)"
              :editing="editing" />

            <ul>
              <li
                v-for="(skill, index) in skills"
                :key="index"
                :contenteditable="editing"
                @input="updateNestedProperty($event, 'skills', index)">
                {{ skill }}
              </li>
            </ul>
            <EditButtons
              @add-click="skills.push('new entry')"
              @remove-click="skills.pop()"
              :show-remove-btn="skills.length > 0" />
          </ResumeSection>

          <ResumeSection>
            <SectionHeadline
              :headline="headlines[3]"
              @headline-edited="updateHeadline($event, 3)"
              :editing="editing" />
            <ul>
              <li
                v-for="(highlight, index) in highlights"
                :key="index"
                :contenteditable="editing"
                @input="updateNestedProperty($event, 'highlights', index)">
                {{ highlight }}
              </li>
            </ul>
            <EditButtons
              @add-click="highlights.push('new entry')"
              @remove-click="highlights.pop()"
              :show-remove-btn="highlights.length > 0" />
          </ResumeSection>
        </div>

        <div class="right-col">
          <div
            class="personal-name"
            :contenteditable="editing"
            @input="updateProperty($event, 'name')">
            {{ name }}
          </div>

          <div
            class="personal-title"
            :contenteditable="editing"
            @input="updateProperty($event, 'title')">
            {{ title }}
          </div>

          <ResumeSection>
            <div class="d-flex">
              <SectionHeadline
                :headline="headlines[4]"
                @headline-edited="updateHeadline($event, 4)"
                :editing="editing" />
              <EditButtons
                :show-remove-btn="false"
                text-add="Add experience"
                @add-click="addExperience" />
            </div>

            <div
              v-for="(item, index) in experience"
              :key="index"
              class="inner-section">
              <div class="d-flex justify-content-between">
                <div
                  :contenteditable="editing"
                  @input="updateExperience($event, 'title', index)">
                  {{ item.title }}
                </div>
                <EditButtons
                  @remove-click="removeExperience(index)"
                  :show-add-btn="false" />
              </div>
              <div class="d-flex justify-content-between">
                <div>
                  <span
                    :contenteditable="editing"
                    @input="updateExperience($event, 'company', index)">
                    {{ item.company }} </span
                  >,
                  <span
                    :contenteditable="editing"
                    @input="updateExperience($event, 'location', index)">
                    {{ item.location }}
                  </span>
                </div>
                <div
                  :contenteditable="editing"
                  @input="updateExperience($event, 'date', index)">
                  {{ item.date }}
                </div>
              </div>

              <ul>
                <li
                  v-for="(desc, innerIndex) in item.description"
                  :key="innerIndex"
                  :contenteditable="editing"
                  @input="updateExperienceDescription($event, index, innerIndex)">
                  {{ desc }}
                </li>
              </ul>
              <EditButtons
                @add-click="item.description.push('new entry')"
                @remove-click="item.description.pop()"
                :show-remove-btn="item.description.length > 0" />
            </div>
          </ResumeSection>

          <ResumeSection>
            <div class="d-flex">
              <SectionHeadline
                :headline="headlines[5]"
                @headline-edited="updateHeadline($event, 5)"
                :editing="editing" />
              <EditButtons
                :show-remove-btn="false"
                text-add="Add education"
                @add-click="addEducation" />
            </div>

            <div
              v-for="(item, index) in education"
              :key="index"
              class="inner-section">
              <div class="d-flex justify-content-between">
                <div
                  :contenteditable="editing"
                  @input="updateEducation($event, 'title', index)">
                  {{ item.title }}
                </div>
                <EditButtons
                  @remove-click="removeEducation(index)"
                  :show-add-btn="false" />
              </div>

              <div class="d-flex justify-content-between">
                <div>
                  <span
                    :contenteditable="editing"
                    @input="updateEducation($event, 'university', index)">
                    {{ item.university }} </span
                  >,
                  <span
                    :contenteditable="editing"
                    @input="updateEducation($event, 'location', index)">
                    {{ item.location }}
                  </span>
                </div>

                <div
                  :contenteditable="editing"
                  @input="updateEducation($event, 'date', index)">
                  {{ item.date }}
                </div>
              </div>

              <ul>
                <li
                  v-for="(desc, innerIndex) in item.description"
                  :key="innerIndex"
                  :contenteditable="editing"
                  @input="updateEducationDescription($event, index, innerIndex)">
                  {{ desc }}
                </li>
              </ul>
              <EditButtons
                @add-click="item.description.push('new entry')"
                @remove-click="item.description.pop()"
                :show-remove-btn="item.description.length > 0" />
            </div>
          </ResumeSection>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import ResumeSection from "./components/ResumeSection.vue";
import SectionHeadline from "./components/SectionHeadline.vue";
import Contact from "./components/Contact.vue";
import EditButtons from "./components/EditButtons.vue";
import ToggleSwitch from "./components/ToggleSwitch.vue";
import Sidebar from "./components/Sidebar.vue";
import ColorInput from "./components/ColorInput.vue";
import PercentageInput from "./components/PercentageInput.vue";
import SelectInput from "./components/SelectInput.vue";
import ImgUpload from "./components/ImgUpload.vue";
import ExportPdf from "./components/ExportPdf.vue";
import CustomButton from "./components/CustomButton.vue";

export default {
  created() {
    const savedResume = localStorage.getItem('resume');
    if(savedResume) {
      try {
        const resume = JSON.parse(savedResume);
        this.loadIntoData(resume);
      } catch(error) {
        console.error("Error parsing saved resume configuration: ", error);
      }
    }
  },
  components: {
    ResumeSection,
    SectionHeadline,
    Contact,
    EditButtons,
    ToggleSwitch,
    Sidebar,
    ColorInput,
    PercentageInput,
    SelectInput,
    ImgUpload,
    ExportPdf,
    CustomButton
  },
  data() {
    return {
      colors: {
        left: {
          highlight: "#82c0cc",
          text: "#ffffff",
          background: "#3943b7",
        },
        right: {
          highlight: "#3943b7",
          text: "#000505",
          background: "#ffffff",
        },
      },
      name: "Michaela Scarn",
      title: "Senior Data Scientist",
      introText:
        "From data cleaning to data analysis to machine learning, I am passionate about everything data.",
      imageUrl: "/profile_pic.jpg",
      headlines: [
        "About me",
        "Contact",
        "Skills",
        "Certifications",
        "Experience",
        "Education",
      ],
      contact: {
        phone: "15713909584",
        email: "contact@gmail.com",
        address: "Main St 100, 19777 NY",
      },
      skills: [
        "Python",
        "Pandas",
        "SQL",
        "R",
        "AI",
        "C++",
        "Machine Learning",
        "Hadoop",
        "TensorFlow",
        "PyTorch",
        "NLP",
      ],
      highlights: [
        "Natural Language Processing with Python (Coursera)",
        "Recommendation Systems with TensorFlow on GCP (Google)",
      ],
      experience: [
        {
          title: "Senior Data Scientist",
          company: "ABC Analytics Inc.",
          location: "London",
          date: "2022 - Present",
          description: [
            "Led a team of data scientists in developing advanced machine learning models for predictive analytics",
            "Designed and implemented a recommendation system that boosted cross-selling, leading to a 20% increase in revenue",
            "Conducted A/B testing and statistical analysis to optimize product features",
          ],
        },
        {
          title: "Data Scientist",
          company: "XYZ Data Solutions",
          location: "London",
          date: "2017 - 2019",
          description: [
            "Developed and deployed machine learning models for fraud detection, reducing fraudulent transactions by 18%",
            "Conducted in-depth exploratory data analysis to identify key trends and insights",
            "Worked on data preprocessing, feature engineering, and model selection to improve model performance",
          ],
        },
        {
          title: "Data Scientist Trainee",
          company: "Data Insights Ltd.",
          location: "New York City",
          date: "2016-2017",
          description: [
            "Collaborated with external partners to integrate third-party data sources, expanding the company's data assets and enhancing predictive modeling capabilities.",
          ],
        },
      ],
      education: [
        {
          title: "Master of Science in Data Science",
          university: "StellarTech University",
          location: "Starville",
          date: "2020-2022",
          description: [
            "Coursework included advanced machine learning, statistical modeling, and data visualization techniques.",
            "Thesis: 'Predictive Modeling for Customer Churn in E-commerce using Random Forest.'",
          ],
        },
        {
          title: "Bachelor of Science in Computer Science",
          university: "Evergreen State University",
          location: "Springdale",
          date: "2012-2015",
          description: [
            "Relevant coursework in database management, algorithms, and programming languages.",
            "Senior project: 'Development of a Recommender System for Movie Ratings.'",
          ],
        },
      ],
      editing: true,
      widthLeft: 30,
      imageShape: "round",
      headlineWeight: "400",
      showImage: true,
      resumeFormat: 'a4'
    };
  },
  computed: {
    cssVariables() {
      return {
        "--highlight-color-left": this.colors.left.highlight,
        "--background-color-left": this.colors.left.background,
        "--text-color-left": this.colors.left.text,
        "--highlight-color-right": this.colors.right.highlight,
        "--background-color-right": this.colors.right.background,
        "--text-color-right": this.colors.right.text,
        "--headline-weight": this.headlineWeight,
      };
    },
    percentageWidthLeft() {
      return this.widthLeft + "%";
    },
  },
  methods: {
    updateHeadline(newValue, index) {
      this.headlines[index] = newValue;
    },
    updateProperty(event, key) {
      this[key] = event.target.innerText;
    },
    updateNestedProperty(event, key1, key2) {
      this[key1][key2] = event.target.innerText;
    },
    updateExperience(event, key, index) {
      this.experience[index][key] = event.target.innerText;
    },
    updateExperienceDescription(event, index1, index2) {
      this.experience[index1]["description"][index2] = event.target.innerText;
    },
    updateEducation(event, key, index) {
      this.education[index][key] = event.target.innerText;
    },
    updateEducationDescription(event, index1, index2) {
      this.education[index1]["description"][index2] = event.target.innerText;
    },
    addExperience() {
      this.experience.unshift({
        title: "Job Title",
        company: "Company",
        location: "Location",
        date: "date range",
        description: ["description"],
      });
    },
    addEducation() {
      this.education.unshift({
        title: "Education title",
        university: "University",
        location: "Location",
        date: "date range",
        description: ["Summa cum laude, GPA 1.0"],
      });
    },
    removeExperience(index) {
      this.experience.splice(index, 1);
    },
    removeEducation(index) {
      this.education.splice(index, 1);
    },
    toggleEditMode(isChecked) {
      this.editing = isChecked;
    },
    toggleImageDisplay(isChecked) {
      this.showImage = isChecked;
    },
    saveConfig() {
      localStorage.setItem('resume', JSON.stringify(this.$data));
    },
    loadIntoData(config) {
      for(const key in config) {
        if(this.$data.hasOwnProperty(key)) {
          this[key] = config[key];
        }
      }
    }
  },
};
</script>

<style scoped>

.resume-wrapper {
  margin-left: auto;
  width: 210mm;
}

#resume {
  box-shadow: rgba(50, 50, 93, 0.25) 0px 13px 27px -5px, rgba(0, 0, 0, 0.3) 0px 8px 16px -8px;
}

#resume.edit-off {
  /* DIN A4 standard paper size commonly used for resumes
  For North America letter size use width: 8.5in; height: 11in; */
  
  /* use 296.8mm height instead of 297mm for a4 to prevent of page break */
  height: 297mm;
}

/* if letter format selected */
#resume.edit-off.letter-format {
  width: 8.5in;
  height: 11in;
}

@media (min-width: 1350px) {
  .resume-wrapper {
    margin-left: 300px;
  }
}

@media (min-width: 1600px) {
  .resume-wrapper {
    margin-left: auto;
    margin-right: auto;
  }
}

.left-col {
  background-color: var(--background-color-left);
  color: var(--text-color-left);
  border-right: 1px solid var(--highlight-color-left);
  padding: 30px;
}

.right-col {
  background-color: var(--background-color-right);
  color: var(--text-color-right);
  width: 70%;
  padding: 30px;
}

.personal-name {
  font-weight: 300;
  color: var(--highlight-color-right);
  font-size: 28px;
  border-bottom: 1px solid var(--highlight-color-right);
  margin: 0;
  margin-left: -30px;
  padding-left: 30px;
  padding-bottom: 15px;
}

.personal-title {
  border-bottom: 1px solid var(--highlight-color-right);
  margin: 0 0 20px -30px;
  padding: 15px 0 15px 30px;
  font-weight: 300;
  font-size: 20px;
}

#resume ul {
  padding-inline-start: 16px;
  margin-block-end: 5px;
  margin-block-start: 5px;
}

.profile-pic {
  display: block;
  width: 160px;
  height: 160px;
  border: 5px solid var(--highlight-color-left);
  margin-bottom: 20px;
  object-fit: cover;
  margin-left: auto;
  margin-right: auto;
}

.circle {
  border-radius: 50%;
}

.inner-section {
  margin-bottom: 20px;
}
</style>
