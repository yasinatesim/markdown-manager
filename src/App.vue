<template>
  <div class="container mx-auto my-16">
    <div v-if="step == 1">
      <textarea class="p-6 placeholder-blue-300 text-dark-600 relative bg-white  text-sm border border-blue-400 outline-none focus:outline-none focus:ring w-full" v-model="template" cols="30" rows="10"></textarea>
      <br /><br />
      <button @click="handleSubmit" class="bg-blue-800 text-white text-sm font-bold  px-6 py-3 rounded shadow hover:shadow-lg outline-none focus:outline-none mr-1 mb-1 w-full ease-linear transition-all duration-150">Submit</button>
    </div>

    <div v-if="step == 2">
      <div v-for="variable in variables">
        <label class="flex items-center py-6 border-b border-gray-300">
          <strong>{{ variable }}</strong
          >:
          <input type="text" v-model="variables[variable]"  class="ml-2 p-2 text-dark-600 relative bg-white text-xs border border-blue-400 outline-none focus:outline-none focus:ring w-full"/>
        </label>

      </div>

      <div v-for="array in arrays" class="mt-4">
        <div class="border-b pb-3 border-gray-300">
          <strong>{{ String(Object.keys(array.values)) }}</strong>

          <div v-for="item in array.values">
            <div v-for="(value, key) in item"> <!-- //  border-b border-gray-300 -->
              <div v-for="obj in Object.keys(value)" class="flex items-center py-6">
                <span>{{ obj }}:</span>
                <input type="text" v-model="value[obj]"  class="ml-2 p-2 text-dark-600 relative bg-white text-xs border border-pink-400 outline-none focus:outline-none focus:ring w-full"/>
              </div>
              <button @click="handleAddInput(item, Object.keys(array.values))" class="bg-pink-800 text-white text-sm font-bold  px-6 py-3 rounded shadow hover:shadow-lg outline-none focus:outline-none mr-1 mb-1 ease-linear transition-all duration-150">
                Add
              </button>
              <button
                @click="handleRemoveInput(Object.keys(array.values), key)"
                v-if="item.length > 1"
              >
                Remove
              </button>
            </div>
          </div>
        </div>
      </div>

      <button @click="handleGetTemplate" class="bg-blue-800 text-white text-sm font-bold  px-6 py-3 mt-6 rounded shadow hover:shadow-lg outline-none focus:outline-none mr-1 mb-1 w-full ease-linear transition-all duration-150">Get Tamplate</button>
    </div>

    <div v-if="step == 3">
      <textarea cols="70" rows="70" readonly class="p-6 placeholder-blue-300 text-dark-600 relative bg-white  text-sm border border-blue-800  w-full">{{ result }}</textarea>
    </div>
  </div>
</template>

<script>
import handlebars from "handlebars";

export default {
  data() {
    return {
      template: `<h3 align="center">
  <br />
   <a  href="{{GITHUB_URL}}"><img src="{{PROJECT_LOGO}}" alt="{{PROJECT_NAME}}" width="200" /></a>
  <br />
{{PROJECT_NAME}}
  <br />
</h3>

<hr />

<p  align="center">{{PROJECT_DESCRIPTION}}</p>


  <p align="center">
{{#links}}
· <a  href="{{link}}">{{label}}</a>
{{/links}}
  </p>

## 📖 About

{{PROJECT_ABOUT}}

### 📚Tech Stack

<table>
{{#techStacks}}
<tr>
  <td> <a href="{{url}}">{{name}}</a></td>
  <td>{{description}}</td>
</tr>
{{/techStacks}}
</table>

## 🧐 What's inside?

{{WHATS_INSIDE}}

## Getting Started

### 📦 Prerequisites

{{PREREQUISITES}}

### ⚙️ How To Use

{{HOW_TO_USE}}

## 🔑 License

* Copyright © {{YEAR}} - {{LICENSE_NAME}} License.

See [LICENSE]({{GITHUB_URL}}/blob/{{GITHUB_MAIN_BRANCH}}/LICENSE) for more information.
`,
      variables: "",
      arrays: [],
      foreachs: [],
      result: "",
      step: 1,
    };
  },

  methods: {
    handleSubmit() {
      this.variables = [
        ...new Set(
          this.template
            .match(/{{([A-Z_]+)+}}/g)
            .map((item) => item.replace(/[\W]+/g, "").trim())
        ),
      ];

      let arraysRegex =
        /({{#(?<arrayName>[a-zA-Z]*)}}(?<content>(.|\n|\t)*?){{\/(.*)}})/gim;

      let match;
      while ((match = arraysRegex.exec(this.template)) != null) {
        const {
          groups: { arrayName, content },
        } = match;

        const foreach = content
          .match(/{{(.*)}}/g)
          .map((item) => {
            return item
              .replace(/{{/g, "")
              .split(/}}/g)
              .map((splittedItem) => {
                return splittedItem.replace(/[^\w]/g, "");
              })
              .filter((item) => item);
          })
          .flat();

        const values = {
          [arrayName]: [
            { ...foreach.reduce((acc, curr) => ((acc[curr] = ""), acc), {}) },
          ],
        };

        this.arrays.push({
          arrayName,
          values,
        });
      }

      this.step++;
    },
    handleGetTemplate() {
      const arrays = this.arrays
        .map((array) => {
          return array.values;
        })
        .reduce((obj, item) => {
          const key = Object.keys(item)[0];
          obj[key] = item[key];
          return obj;
        }, {});

      const template = handlebars.compile(this.template);
      this.result = template({ ...this.variables, ...arrays });

      this.step++;
    },
    handleAddInput(key, array) {
      const findedArray = this.arrays.find((item) => {
        return item.values[array];
      });

      const obj = Object.keys(Object.values(key)[0]).reduce(
        (acc, curr) => ((acc[curr] = ""), acc),
        {}
      );

      findedArray.values[array].push(obj);
    },
    handleRemoveInput(array, index) {
      const findedArray = this.arrays.find((item) => {
        return item.values[array];
      });

      findedArray.values[array].splice(index, 1);
    },
  },
};
</script>
