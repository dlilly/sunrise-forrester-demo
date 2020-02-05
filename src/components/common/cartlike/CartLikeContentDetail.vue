<template>
  <div>
    <div class="row">
      <div class="row text-uppercase hidden-xs cart-items-title">
        <div class="col-sm-6">
          <span>{{ $t('description') }}</span>
        </div>
        <div class="col-sm-2">
          <span>{{ $t('quantity') }}</span>
        </div>
        <div class="col-sm-2 text-right">
          <span>{{ $t('price') }}</span>
        </div>
        <div class="col-sm-2 text-right">
          <span>{{ $t('total') }}</span>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-12">
      <div v-for="lineItem in cartLike.lineItems"
           :key="lineItem.id"
           data-test="cart-line-item"
           class="row">
           <div class="row single-cart-item">
        <LineItemInfo :line-item="lineItem"
                      class="col-sm-4 col-xs-12"/>
        <div class="col-sm-4 col-xs-12">
          <div v-if="editable">
            <LineItemDeleteForm :lineItemId="lineItem.id"
                                class="col-sm-5 cart-edit-delete"/>
            <LineItemQuantityForm :lineItemId="lineItem.id"
                                  :quantity="lineItem.quantity"
                                  class="col-sm-7 clearfix sm-pull-right"/>
          </div>
          <div v-else
               class="col-sm-6 col-sm-offset-6 col-xs-12 text-center quantity-counter">
            <span class="visible-xs">{{ $t('quantity') }}:</span>
            <span data-test="cart-line-item-quantity"
                  class="quantity-number">
              {{ lineItem.quantity }}
            </span>
          </div>
        </div>
        <div>
          <div class="col-sm-2 col-xs-12 sm-pull-right">
            <div class="text-right cart-item-price">
              <span class="visible-xs xs-price-title">{{ $t('price') }}</span>
              <BasePrice :price="lineItem.price"/>
            </div>
          </div>
          <div class="col-sm-2 col-xs-12 sm-pull-right">
            <div class="text-right cart-item-price">
              <span class="visible-xs xs-price-title">{{ $t('total') }}</span>
              <span data-test="cart-line-item-total-price">
                <BasePrice :price="totalPrice(lineItem)"/>
              </span>
            </div>
          </div>
        </div>
        </div>
        <div class="row discount-detail">
          <div class="col-md-6"></div>
          <div class="col-md-6">
            <div v-for="discount in groupedDiscounts(lineItem)"
              :key="discount.name">
              <div v-if="discount.discount.centAmount > 0" class="row">
                <dt class="col-md-9">{{ discount.name }}</dt>
                <dd class="col-md-3 text-right">
                  <BaseMoney
                    :money="discount.discount"
                    :negate="true"/>
                </dd>
              </div>
            </div>
          </div>

          <!-- <div class="col-md-6">
            <div v-for="discountPerQuantity in lineItem.discountedPricePerQuantity"
              :key="discountPerQuantity.id">
              <div v-for="discount in discountPerQuantity.discountedPrice.includedDiscounts"
                :key="discount.name">
                <div v-if="discount.discountedAmount.centAmount > 0" class="row">
                  <dt class="col-md-9">{{ discount.discount.name }}</dt>
                  <dd class="col-md-3">
                    <BaseMoney
                      :money="discount.discountedAmount"
                      :quantity="discountPerQuantity.quantity"
                      :negate="true"/>
                  </dd>
                </div>
              </div>
            </div>
          </div> -->
        </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import LineItemInfo from './LineItemInfo.vue';
import BasePrice from '../BasePrice.vue';
import BaseMoney from '../BaseMoney.vue';
import LineItemQuantityForm from '../../cartdetail/LineItemQuantityForm.vue';
import LineItemDeleteForm from '../../cartdetail/LineItemDeleteForm.vue';

export default {
  components: {
    LineItemDeleteForm,
    LineItemQuantityForm,
    BasePrice,
    BaseMoney,
    LineItemInfo,
  },

  props: {
    cartLike: {
      type: Object,
      required: true,
    },
    editable: {
      type: Boolean,
      default: false,
    },
  },

  methods: {
    totalPrice(lineItem) {
      const { centAmount: unitCentAmount, ...unitPrice } = lineItem.price.discounted?.value || lineItem.price.value;
      const originalPrice = {
        ...unitPrice,
        centAmount: unitCentAmount * lineItem.quantity,
      };
      const price = { value: { ...originalPrice } };
      if (originalPrice.centAmount !== lineItem.totalPrice.centAmount) {
        price.discounted = { value: { ...lineItem.totalPrice } };
      }
      return price;
    },

    groupedDiscounts(lineItem) {
      const discountHash = {};
      lineItem.discountedPricePerQuantity.forEach((element) => {
        const discount = element.discountedPrice;

        discount.includedDiscounts.forEach((d) => {
          let discountSummary = discountHash[d.discount.name];

          if (!discountSummary) { // so, first time through the loop
            discountSummary = {
              name: d.discount.name,
              discount: {
                centAmount: 0,
                currencyCode: d.discountedAmount.currencyCode,
                fractionDigits: d.discountedAmount.fractionDigits,
                __typename: d.discountedAmount.__typename,
              },
            };
          }

          discountSummary.discount.centAmount += d.discountedAmount.centAmount * element.quantity;
          discountHash[d.discount.name] = discountSummary;
        });
      });

      return Object.values(discountHash);
    },
  },
};
</script>

<i18n>
en:
  description: "Description"
  quantity: "Quantity"
  price: "Price"
  total: "Total"
de:
  description: "Beschreibung"
  quantity: "Menge"
  price: "Preis"
  total: "Gesamtpreis"
</i18n>
