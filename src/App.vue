<template>
  <div v-if="step == 1">
    <textarea v-model="template" cols="30" rows="10"></textarea>
    <br /><br />
    <button @click="handleSubmit">Submit</button>
  </div>

  <div v-if="step == 2">
    <div v-for="variable in variables">
      <label>
        <strong>{{ variable }}</strong
        >:
        <input type="text" v-model="variables[variable]" />
      </label>
      <br />
      <br />
    </div>

    {{ arrays }}

    <div v-for="array in arrays">
      <div>
        <strong>{{ Object.keys(array) }}</strong>

        <div v-for="item in array">
          <div v-for="(value,key) in item">
            <div v-for="obj in Object.keys(value)">
              <span>{{ obj }}:</span>
              <input type="text" v-model="value[obj]" />
            </div>
            <button @click="handleAddInput(item, Object.keys(array))">
              Add
            </button>
            <button @click="handleRemoveInput(item, Object.keys(array), key)" v-if="item.length > 1">
              Remove
            </button>
          </div>
        </div>
      </div>
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
      arrays: [],
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

      let arraysRegex =
        /<%\s\@foreach\s(?<arrayKey>[a-zA-Z]*)\sin\s(?<arrayName>[a-zA-Z]*)\s%>(.|\n|\t)*?<%\s\@endforeach\s%>/gim;

      let match;
      while ((match = arraysRegex.exec(this.template)) != null) {
        const {
          groups: { arrayName },
        } = match;

        const foreach = match[0]
          .match(/<%=\s(.*)\s=%>/g)
          .map((item) => {
            return item
              .replace(/<%=\s/g, "")
              .split(/\s=%>/g)
              .map((splittedItem) => {
                return splittedItem.replace(/[^\w.]/g, "");
              })
              .filter((item) => item);
          })
          .flat();

        this.arrays.push({
          [arrayName]: [
            { ...foreach.reduce((acc, curr) => ((acc[curr] = ""), acc), {}) },
          ],
        });
      }

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
    handleAddInput(key, array) {
      const findedArray = this.arrays.find((item) => {
        return item[array];
      });

      const obj = Object.keys(Object.values(key)[0]).reduce(
        (acc, curr) => ((acc[curr] = ""), acc),
        {}
      );

      findedArray[array].push(obj);
    },
    handleRemoveInput(key, array, index) {
      console.log('key:', key)
      console.log('index:', index)
      const findedArray = this.arrays.find((item) => {
        return item[array];
      });

      findedArray[array].splice(index, 1);
    },
  },
};
</script>
