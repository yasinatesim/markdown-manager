<script>
import handlebars from 'handlebars';

// Components
import Alert from './components/alert.vue';
import Button from './components/button.vue';

export default {
  data() {
    return {
      template: '',
      variables: [],
      arrays: [],
      result: '',
      step: 1,
    };
  },
  components: {
    Alert,
    Button,
  },
  methods: {
    copyDoiToClipboard(str) {
      navigator.clipboard.writeText(str);
    },
    handleSubmit() {
      this.variables = [
        ...new Set(this.template.match(/{{([A-Z_]+)+}}/g).map((item) => item.replace(/[\W]+/g, '').trim())),
      ];

      let arraysRegex = /({{#(?<arrayName>[a-zA-Z]*)}}(?<content>(.|\n|\t)*?){{\/(.*)}})/gim;

      let match;
      while ((match = arraysRegex.exec(this.template)) != null) {
        const {
          groups: { arrayName, content },
        } = match;

        const foreach = content
          .match(/{{(.*)}}/g)
          .map((item) => {
            return item
              .replace(/{{/g, '')
              .split(/}}/g)
              .map((splittedItem) => {
                return splittedItem.replace(/[^\w]/g, '');
              })
              .filter((item) => item);
          })
          .flat();

        const values = {
          [arrayName]: [{ ...foreach.reduce((acc, curr) => ((acc[curr] = ''), acc), {}) }],
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

      this.copyDoiToClipboard(this.result);

      this.step++;
    },
    handleAddInput(key, array) {
      const findedArray = this.arrays.find((item) => {
        return item.values[array];
      });

      const obj = Object.keys(Object.values(key)[0]).reduce((acc, curr) => ((acc[curr] = ''), acc), {});

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

<template>
  <div class="container mx-auto my-16">
    <div v-if="step == 1">
      <textarea
        class="
          p-6
          placeholder-blue-300
          text-dark-600
          relative
          bg-white
          text-sm
          border border-blue-400
          outline-none
          focus:outline-none focus:ring
          w-full
        "
        v-model="template"
        cols="30"
        rows="10"
      ></textarea>
      <br /><br />
      <Button @click="handleSubmit" variant="primary" class="mr-1 mb-1 w-full"> Submit </Button>
    </div>

    <div v-if="step == 2">
      <div v-for="variable in variables">
        <label class="flex items-center py-6 border-b border-gray-300">
          <strong>{{ variable }}</strong
          >:
          <textarea
            type="text"
            v-model="variables[variable]"
            class="
              ml-2
              p-2
              text-dark-600
              relative
              bg-white
              text-xs
              border border-blue-400
              outline-none
              focus:outline-none focus:ring
              w-full
            "
          ></textarea>
        </label>
      </div>

      <div v-for="array in arrays" class="mt-4">
        <div class="border-b pb-3 border-gray-300">
          <strong>{{ String(Object.keys(array.values)) }}</strong>

          <div v-for="item in array.values">
            <div v-for="(value, key) in item">
              <div v-for="obj in Object.keys(value)" class="flex items-center py-6">
                <span>{{ obj }}:</span>
                <input
                  type="text"
                  v-model="value[obj]"
                  class="
                    ml-2
                    p-2
                    text-dark-600
                    relative
                    bg-white
                    text-xs
                    border border-pink-400
                    outline-none
                    focus:outline-none focus:ring
                    w-full
                  "
                />
              </div>
              <Button @click="handleAddInput(item, Object.keys(array.values))" class="mr-1 mb-1" variant="success">
                Add
              </Button>
              <Button
                @click="handleRemoveInput(Object.keys(array.values), key)"
                v-if="item.length > 1"
                class="mr-1mb-1"
                variant="danger"
              >
                Remove
              </Button>
            </div>
          </div>
        </div>
      </div>

      <Button @click="handleGetTemplate" variant="primary" class="mr-1 mb-1 w-full"> Get Template </Button>
    </div>

    <div v-if="step == 3">
      <Alert />
      <div>
        <pre
          cols="70"
          rows="70"
          readonly
          class="rıunded p-6 text-dark-600 relative bg-white text-sm border border-blue-800 w-full"
        >
          <code>{{ result }}</code></pre>
      </div>
    </div>
  </div>
</template>
