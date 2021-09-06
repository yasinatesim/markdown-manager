<template>
  <div v-if="step == 1">
    <textarea v-model="template" cols="30" rows="10"></textarea>
    <br /><br />
    <button @click="handleSubmit">Submit</button>
  </div>

  {{ arrays }}

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

    <div v-for="array in arrays">
      <div>
        <strong>{{ String(Object.keys(array.values)) }}</strong>

        <div v-for="item in array.values">
          <div v-for="(value, key) in item">
            <div v-for="obj in Object.keys(value)">
              <span>{{ obj }}:</span>
              <input type="text" v-model="value[obj]" />
            </div>
            <button @click="handleAddInput(item, Object.keys(array.values))">
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
      template: `<h3 align="center">
  <br />
   <a  href="<%= GITHUB_URL =%>"><img src="<%= PROJECT_LOGO =%>" alt="<%= PROJECT_NAME =%>" width="200" /></a>
  <br />
<%= PROJECT_NAME =%>
  <br />
</h3>

<hr />

<p  align="center"><%= PROJECT_DESCRIPTION =%></p>


  <p align="center">
<% @foreach item in links %>
· <a  href="<%= item.link =%>"><%= item.label =%></a>
<% @endforeach %>
  </p>

## 📖 About

<%= PROJECT_ABOUT =%>

### 📚Tech Stack

<table>
<% @foreach tech in techStacks %>
  <tr>
    <td> <a href="<%= tech.url =%>"><%= tech.name =%></a></td>
    <td><%= tech.description =%></td>
  </tr>
<% @endforeach %>
</table>

## 🧐 What's inside?

<%= WHATS_INSIDE =%>

## Getting Started

### 📦 Prerequisites

<%= PREREQUISITES =%>

### ⚙️ How To Use

<%= HOW_TO_USE =%>

## 🔑 License

* Copyright © <%= YEAR =%> - <%= LICENSE_NAME =%> License.

See [LICENSE](<%= GITHUB_URL =%>/blob/<%= GITHUB_MAIN_BRANCH =%>/LICENSE) for more information.
`,
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
        /(?<startForeach><%\s\@foreach\s(?<arrayKey>[a-zA-Z]*)\sin\s(?<arrayName>[a-zA-Z]*)\s%>)(?<content>(.|\n|\t)*?)(?<endForeach><%\s\@endforeach\s%>)/gim;

      let match;
      while ((match = arraysRegex.exec(this.template)) != null) {
        const {
          groups: { startForeach, endForeach, arrayKey, arrayName, content },
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

        const values = {
          [arrayName]: [
            { ...foreach.reduce((acc, curr) => ((acc[curr] = ""), acc), {}) },
          ],
        };

        this.arrays.push({
          startForeach,
          endForeach,
          arrayKey,
          arrayName,
          content,
          values,
          match: match[0],
        });
      }

      this.step++;
    },
    handleGetTemplate() {
      const variableKeys = Object.keys(this.variables);

      // arrays filter with foreachs same item

      // const startForEachs = this.foreachs.map(foreach => foreach.startForeach);
      // const endForEachs = this.foreachs.map(foreach => foreach.endForeach);

      const variablesRegexStr = variableKeys
        .map((item) => {
          return `<%=\\s(${item})\\s=%>`;
        })
        .join("|");

      // const startForeachStr = startForEachs.join("|");
      // const endForEachStr = endForEachs.join("|");

      const variablesRegex = new RegExp(variablesRegexStr, "g");
      this.result = this.template.replace(variablesRegex, (matched) => {
        return this.variables[matched.replace(/<%=|=%>/g, "").trim()];
      });

      // const startForeach = new RegExp(/<%=\s([a-zA-Z.]+)\s=%>/, "g");

      // const startForeach = new RegExp(startForeachStr, "g");
      // this.result = this.result.replace(startForeach, `console.log('deneme')`);

      // https://www.codegrepper.com/code-examples/javascript/js+execute+string
      // https://krasimirtsonev.com/blog/article/Javascript-template-engine-in-just-20-line
      // https://github.com/krasimir/absurd/blob/master/lib/processors/html/helpers/TemplateEngine.js
      //https://blog.stevenlevithan.com/archives/javascript-match-nested

      // const startForeach = new RegExp(startForeachStr, "g");
      // this.result = this.result.replace(startForeach, `console.log('deneme')`);

      // const endForeach = new RegExp(endForEachStr, "g");
      // this.result = this.result.replace(endForeach, '');

      this.arrays.forEach((array) => {
        array.values[array.arrayName].forEach((item, index) => {
          Object.keys(item).forEach((key) => {
            const regex = new RegExp(`<%=\\s${key}\\s=%>`, "g");
            this.result = this.result.replace(regex, (matched) => {
              return item[key];
            });
          });
        });
      });

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
