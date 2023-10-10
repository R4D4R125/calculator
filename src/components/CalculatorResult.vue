<template>
  <div class="result" v-if="insurancePrice !== null">
    <p>Cena poistenia: {{ insurancePrice.toFixed(2) }} €</p>
  </div>
</template>

<script>
export default {
  props: {
    insurancePrice: Number,
  },
};
</script>

<style>
.result {
  font-weight: bold;
  text-align: center;
  font-size: 40px;
}
</style>