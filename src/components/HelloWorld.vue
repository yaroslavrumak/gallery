<template>
  <b-container>
    <b-form inline class="mt-4">
      <label class="sr-only" for="select">Filter primary color</label>
      <b-form-select
        v-model="selected"
        id="select"
        :options="options"
      ></b-form-select>
    </b-form>
  </b-container>
</template>

<script>
export default {
  name: "HelloWorld",

  data() {
    return {
      selected: null,
      options: [
        { value: null, text: "Please select an option" },
        { value: "red", text: "Red" },
        { value: "blue", text: "Blue" },
        { value: "green", text: "Green" },
        { value: "none", text: "None" },
      ],
    };
  },
};
</script>
