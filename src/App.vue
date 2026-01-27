<template>
  <div>
    <csv-editor :data="data" />
  </div>
</template>

<script>
import { ref } from 'vue';
import CsvEditor from "./components/CsvEditor.vue";
//import CsvEditor from "./components/ReadLog.vue";

export default {
  name: "App",
  components: {
    CsvEditor,
  },
  setup() {
    let data = ref("");
    data.value = "name,role,country \n Sarene,Help Desk Operator,Thailand \n Olvan,Nurse Practicioner,China \n Janos,Cost Accountant,China \n Dolph,Assistant Manager,China \n Ariela,Database Administrator I,Azerbaijan \n Lane,Environmental Tech,Indonesia \n Griselda,Senior Quality Engineer,Portugal \n Manda,Physical Therapy Assistant,Brazil \n Leslie,Information Systems Manager,Japan \n Aleen,Cost Accountant,Canada"
    return {data}
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
