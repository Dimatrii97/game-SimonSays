<template>
  <div>
    <label v-for="(title, value) in map" :key="value">
      <input
        :value="value"
        :checked="active == value"
        :name="name"
        @input="$emit('input', value)"
        type="radio"
      />{{ title }}<br />
    </label>
  </div>
</template>

<script>
export default {
  props: {
    map: Object,
    active: [String, Number],
    name: String
  }
};
</script>

<style></style>
