<template>
  <span data-test="product-original-price">
    {{ formattedMoney }}
  </span>
</template>

<script>
export default {
  props: {
    money: Object,
    quantity: Number,
    negate: Boolean,
  },

  computed: {
    formattedMoney() {
      return this.$n(this.amount, 'currency', this.$store.state.country);
    },

    amount() {
      if (this.money) {
        return this.money.centAmount / (10 ** this.money.fractionDigits)
              * (this.quantity || 1)
              * (this.negate ? -1 : 1);
      }
      return 0;
    },
  },
};
</script>
