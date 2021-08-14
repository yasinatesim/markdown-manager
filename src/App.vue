<template>
  <div v-if="step == 1">
    <textarea v-model="template" cols="30" rows="10"></textarea>
    <br /><br />
    <button @click="handleSubmit">Submit</button>
  </div>

<div v-if="step == 2" >
  <div v-for="variable in variables">
    <label>
      <strong>{{ variable }}</strong>:
      <input type="text" :name="variable" v-model="variables[variable]" />
      {{ variables[variable] }}
    </label>
    <br>
    <br>
  </div>
    <button @click="handleGetTemplate">Get Tamplate</button>
</div>

  <div v-if="step == 3">
    <textarea cols="70" rows="70" readonly>{{ result }}</textarea>
  </div>
</template>

<script>
export default {
  data() {
    return {
      template: "",
      variables: "",
      arrays: "",
      result: "",
      step: 1,
    };
  },

  methods: {
    handleSubmit() {
      this.variables = [
        ...new Set(
          this.template
            .match(/<%=\s([A-Z_]+)+\s=%>/g)
            .map((item) => item.replace(/[\W]+/g, "").trim())
        ),
      ];
      this.step++;
    },
    handleGetTemplate() {
      const variableKeys = Object.keys(this.variables);

      const variablesRegexStr = variableKeys
        .map((item) => {
          return `<%=\\s(${item})\\s=%>`;
        })
        .join("|");

      const variablesRegex = new RegExp(variablesRegexStr, "g");
      this.result = this.template.replace(variablesRegex, (matched) => {
        return this.variables[matched.replace(/<%=|=%>/g, "").trim()];
      });

      this.step++;
    },
  },
};
</script>
