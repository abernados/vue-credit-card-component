<template>
  <div class="credit-card">
    <form @submit.prevent="validateCard" class="my-5 bg-gray-100 p-2 rounded-lg text-center">
      <fieldset>
        <legend class="block text-sm font-medium leading-6 text-gray-900">Enter you card details</legend>
        <div class="mt-2 -space-y-px rounded-md bg-white shadow-sm">
          <div class="relative mt-2 rounded-md shadow-sm">
            <input v-model="cardDetails.number" 
                  v-imask="cardNumberMaskOptions" 
                  placeholder="Card number" 
                  :class="{'text-red-500': cardErrors.number && cardDetails.number !== ''}"
                  class="px-2 block w-full rounded-none rounded-t-md border-0 py-1.5 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6"/>
            <div class="pointer-events-none absolute inset-y-0 right-2 flex items-center pl-3">
              <img class="h-6 w-6" 
                  :src="selectedCard.icon" 
                  :alt="selectedCard.name" 
                  v-if="selectedCard && selectedCard.icon" />
              <CreditCardIcon class="h-6 w-6 text-gray-400" :class="{'text-red-500' : cardErrors.number}" aria-hidden="true" v-else/>
            </div>
          </div>
          <div class="flex -space-x-px">
            <div class="w-1/2 min-w-0 flex-1">
              <label for="card-expiration-date" class="sr-only">Expiration date</label>
              <input v-model="cardDetails.expiry" 
                    v-imask="cardExpiryMaskOptions" 
                    placeholder="MM / YY or YYYY" 
                    :class="{'text-red-500': cardErrors.expiry}"
                    class="px-2 relative block w-full rounded-none rounded-bl-md border-0 bg-transparent py-1.5 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:z-10 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6" />
            </div>
            <div class="min-w-0 flex-1">
              <label for="card-cvc" class="sr-only">CVC</label>
              <input v-model="cardDetails.cvv" 
                    v-imask="cardCvvMaskOptions" 
                    class="px-2 relative block w-full rounded-none rounded-br-md border-0 bg-transparent py-1.5 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:z-10 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6" 
                    placeholder="CVC" />
            </div>
          </div>
        </div>
      </fieldset>
      <button type="submit" 
              :disabled="!(cardDetails.number && cardDetails.expiry && cardDetails.cvv && cardDetails.type)" 
              :class="{'opacity-50 cursor-not-allowed	': !(cardDetails.number && cardDetails.expiry && cardDetails.cvv && cardDetails.type)}"
              class="inline-flex items-center gap-x-1.5 rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">
        <LockClosedIcon class="-ml-0.5 h-5 w-5" aria-hidden="true" />
        Pay
      </button>   
    </form>
    <div class="rounded-md bg-red-50 p-4" v-if="Object.keys(cardErrors).length !== 0">
      <div class="flex">
        <div class="flex-shrink-0">
          <XCircleIcon class="h-5 w-5 text-red-400" aria-hidden="true" />
        </div>
        <div class="ml-3">
          <h3 class="text-sm font-medium text-red-800">There's an error with your submission</h3>
          <div class="mt-2 text-sm text-red-700">
            <ul role="list" class="list-disc space-y-1 pl-5">
              <li v-for="(error, field) in cardErrors" :key="field">
                 {{ error }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
    <div class="rounded-md bg-green-50 p-4" v-else>
      <div class="flex">
        <div class="flex-shrink-0">
          <CheckCircleIcon class="h-5 w-5 text-green-400" aria-hidden="true" />
        </div>
        <div class="ml-3">
          <p class="text-sm font-medium text-green-800">Successfully sent your payment</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch, computed  } from 'vue';
import { IMaskDirective } from 'vue-imask';
import IMask from 'imask';
import { CreditCardIcon, LockClosedIcon, XCircleIcon, CheckCircleIcon } from '@heroicons/vue/20/solid'

export default {
  name: 'CreditCard',
  directives: {
    imask: IMaskDirective,
  },
  components: {
    CreditCardIcon,
    LockClosedIcon,
    XCircleIcon,
    CheckCircleIcon, 
  },
  setup() {
    const valid = ref(false);
    const cardNumberMaskOptions = ref({
      mask: '0000 0000 0000 0000',
      lazy: false,
      overwrite: 'shift',
    });
    
    const cardExpiryMaskOptions = ref({
      autofix: true,
      mask: 'M/Y',
      blocks: {
        M: {mask: IMask.MaskedRange, from: 1, to: 12, maxLength: 2},
        Y: {mask: IMask.MaskedRange, from: 20, to: 2999, minLength: 2,maxLength: 4}
      },

    });
    
    const cardCvvMaskOptions = ref({
      mask: '0000',
      min: 100,
      max: 9999,
    });

    const cardDetails = ref({
      number: '',
      expiry: '',
      cvv: '',
      type: '',
    });

    const cardErrors = ref({});
    const selectedCard = ref({
      name: '',
      icon: '',
    });

    const cardNumberComputed = computed(() => cardDetails.value.number);
    const isCardExpired = computed(() => cardDetails.value.expiry);

    const updateMask = () => {
      const cardNumber = cardDetails.value.number.replace(/\s/g, '');

      const cardType = getCardType(cardNumber);

      if(cardType){
        selectedCard.value.icon = cardType.icon;
        cardDetails.value.type = cardType.name;

        if (cardType.name === 'American Express') {
          cardNumberMaskOptions.value.mask = '0000 000000 00000';
        }else{
          cardNumberMaskOptions.value.mask = '0000 0000 0000 0000';
        }

        if (!validateCardNumber(cardNumber)) {
          cardErrors.value.number = 'Invalid card number.';
          selectedCard.value.icon = null;
        }else{
        delete cardErrors.value['number'];

        }

      }else{
        cardErrors.value.number = 'Invalid card number.';
        selectedCard.value.icon = null;
      }
      
    };

    const validateCard = () => { 
      console.log(cardErrors.value,Object.keys(cardErrors.value), Object.keys(cardErrors.value).length, Object.keys(cardErrors.value).length === 0);     
      if (Object.keys(cardErrors.value).length === 0) {
        valid.value = true;
      } else {
        valid.value = false;
      }
      console.log('valid: ', valid.value);
    };

    const getCardType = (cardNumber) => {
      const cardTypes = [
        {
          name: 'Visa',
          pattern: /^4/,
          icon: 'https://img.icons8.com/color/48/visa.png'
        },
        {
          name: 'Mastercard',
          pattern: /^5[1-5]/,
          icon: 'https://img.icons8.com/color/48/mastercard.png'
        },
        {
          name: 'American Express',
          pattern: /^3[47]/,
          icon: 'https://img.icons8.com/color/48/amex.png'
        },
        {
          name: 'Discover',
          pattern: /^6(?:011|5)/,
          icon: 'https://img.icons8.com/color/48/discover.png'
        },
        {
          name: 'JCB',
          pattern: /^(?:2131|1800|35)/,
          icon: 'https://img.icons8.com/color/48/jcb.png'
        },
        {
          name: 'Diners Club',
          pattern: /^3(?:0[0-5]|[68])/,
          icon: 'https://img.icons8.com/color/48/diners-club.png'
        },
      ];

      for (const cardType of cardTypes) {
        if (cardNumber.match(cardType.pattern)) {
          return cardType;
        }
      }

      return null;
    };

    const validateCardNumber = (cardNumber) => {
      const strippedNumber = cardNumber.replace(/\D/g, '');
      if (!luhnAlgorithm(strippedNumber)) {
        return false;
      }

      return true;
    };

    const validateExpiry = () => {
      const expiry = cardDetails.value.expiry;
      const pattern = /^(0[1-9]|1[0-2])\/(20\d{2}|[3-9]\d)$/;

      if (!pattern.test(expiry)) {
        cardErrors.value.expiry = 'Invalid expiry date.';
      }

      let [month, year] = expiry.split('/');
      year = new Date(parseInt(year), 10).getFullYear()% 100;
      
      const currentDate = new Date();
      const currentYear = currentDate.getFullYear() % 100;
      const currentMonth = currentDate.getMonth() + 1;
    
      if (year < currentYear) {
        cardErrors.value.expiry = 'Invalid expiry date.';

      } else if (year === currentYear && parseInt(month, 10) < currentMonth) {
        cardErrors.value.expiry = 'Invalid expiry date.';
      }else{
        delete cardErrors.value['expiry'];
      }
    };

    const luhnAlgorithm = (cardNumber) => {
      let sum = 0;
      let isSecondDigit = false;
      for (let i = cardNumber.length - 1; i >= 0; i--) {
        let digit = parseInt(cardNumber.charAt(i), 10);
        if (isSecondDigit) {
          digit *= 2;
          if (digit > 9) {
            digit -= 9;
          }
        }
        sum += digit;
        isSecondDigit = !isSecondDigit;
      }
      return sum % 10 === 0;
    };

    watch(cardNumberComputed, updateMask);
    watch(isCardExpired, validateExpiry);

    return {
      valid,
      cardDetails,
      cardErrors,
      validateCard,
      updateMask,
      cardNumberMaskOptions,
      cardExpiryMaskOptions,
      cardCvvMaskOptions,
      selectedCard,
      validateExpiry,
    };
  },
};
</script>

<style>
.credit-card {
  font-family: Arial, sans-serif;
  max-width: 400px;
  margin: 0 auto;
}

h2 {
  text-align: center;
}

form {
  display: grid;
  grid-gap: 10px;
}

label {
  font-weight: bold;
}

input[type="text"],
input[type="number"] {
  width: 100%;
  padding: 5px;
}

button {
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

.error {
  color: red;
  font-size: 12px;
  margin-top: 5px;
}
</style>

