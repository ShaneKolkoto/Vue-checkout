<template>
 <div class="col-span-1 lg:col-span-6">
  <h4 class="text-3xl text-gray-700 mb-5">Payment information</h4>
  <div class="p-10 rounded-md shadow-md bg-white">
   <div v-if="cardComponent" class="mb-6">
    <Card
     :fields="fields"
     :labels="formData"
     :isCardNumberMasked="isCardNumberMasked"
     :randomBackgrounds="randomBackgrounds"
     :backgroundImage="backgroundImage"
    />
   </div>
   <div class="mb-6">
    <label class="block mb-3 text-gray-600" for="">Name on card</label>
    <input
     type="text"
     :id="fields.cardName"
     v-letter-only
     @input="changeName"
     :value="formData.cardName"
     data-card-field
     autocomplete="off"
     class="border border-gray-500 rounded-md inline-block py-2 px-3 w-full text-gray-600 tracking-wider"
    />
   </div>
   <div class="mb-6">
    <label class="block mb-3 text-gray-600" for="">Card number</label>
    <input
     type="tel"
     :id="fields.cardNumber"
     @input="changeNumber"
     @focus="focusCardNumber"
     @blur="blurCardNumber"
     :value="formData.cardNumber"
     :maxlength="cardNumberMaxLength"
     data-card-field
     autocomplete="off"
     class="border border-gray-500 rounded-md inline-block py-2 px-3 w-full text-gray-600 tracking-widest"
    />
   </div>
   <div class="mb-6 flex flex-wrap -mx-3w-full">
    <div class="w-2/3 px-3">
     <label class="block mb-3 text-gray-600" for="">Expiraion date</label>
     <div class="flex">
      <select
       class="border border-gray-500 rounded-md inline-block py-2 px-3 w-full text-gray-600 tracking-widest mr-6"
       :id="fields.cardMonth"
       v-model="formData.cardMonth"
       @change="changeMonth"
       data-card-field
      >
       <option value disabled selected>Month</option>
       <option
        v-bind:value="n < 10 ? '0' + n : n"
        v-for="n in 12"
        v-bind:disabled="n < minCardMonth"
        v-bind:key="n"
        >{{ generateMonthValue(n) }}</option
       >
      </select>
      <select
       class="border border-gray-500 rounded-md inline-block py-2 px-3 w-full text-gray-600 tracking-widest"
       :id="fields.cardYear"
       v-model="formData.cardYear"
       @change="changeYear"
       data-card-field
      >
       <option value disabled selected>Year</option>
       <option
        v-bind:value="$index + minCardYear"
        v-for="(n, $index) in 12"
        v-bind:key="n"
        >{{ $index + minCardYear }}</option
       >
      </select>
     </div>
    </div>
    <div class="w-1/3 px-3">
     <label class="block mb-3 text-gray-600" for="">CVC</label>
     <input
      type="tel"
      v-number-only
      :id="fields.cardCvv"
      maxlength="4"
      :value="formData.cardCvv"
      @input="changeCvv"
      data-card-field
      autocomplete="off"
      class="border border-gray-500 rounded-md inline-block py-2 px-3 w-full text-gray-600 tracking-widest"
     />
    </div>
   </div>
   <div class="mb-6 text-right">
    <span class="text-right font-bold">{{ total }} ZAR</span>
   </div>
   <div>
    <button @click="finishPayment"
     class="w-full text-ceenter px-4 py-3 bg-blue-500 rounded-md shadow-md text-white font-semibold"
    >
     Confirm payment
    </button>
   </div>
  </div>
 </div>
</template>
<script>
import Card from "./Card";
export default {
  name: "Payment",
  directives: {
    "number-only": {
      bind(el) {
        function checkValue(event) {
          event.target.value = event.target.value.replace(/[^0-9]/g, "");
          if (event.charCode >= 48 && event.charCode <= 57) {
            return true;
          }
          event.preventDefault();
        }
        el.addEventListener("keypress", checkValue);
      }
    },
    "letter-only": {
      bind(el) {
        function checkValue(event) {
          if (event.charCode >= 48 && event.charCode <= 57) {
            event.preventDefault();
          }
          return true;
        }
        el.addEventListener("keypress", checkValue);
      }
    }
  },
  props: {
    total: Number,
    formData: {
      type: Object,
      default: () => {
        return {
          cardName: "",
          cardNumber: "",
          cardMonth: "",
          cardYear: "",
          cardCvv: ""
        };
      }
    },
    backgroundImage: [String, Object],
    randomBackgrounds: {
      type: Boolean,
      default: true
    }
  },
  components: {
    Card
  },
  data() {
    return {
      cardComponent: true,
      fields: {
        cardNumber: "v-card-number",
        cardName: "v-card-name",
        cardMonth: "v-card-month",
        cardYear: "v-card-year",
        cardCvv: "v-card-cvv"
      },
      minCardYear: new Date().getFullYear(),
      isCardNumberMasked: true,
      mainCardNumber: this.cardNumber,
      cardNumberMaxLength: 19
    };
  },
  computed: {
    minCardMonth() {
      if (this.formData.cardYear === this.minCardYear)
        return new Date().getMonth() + 1;
      return 1;
    }
  },
  watch: {
    cardYear() {
      if (this.formData.cardMonth < this.minCardMonth) {
        this.formData.cardMonth = "";
      }
    }
  },
  mounted() {
    this.maskCardNumber();
    if (this.cardComponent === true) {
      this.$emit("handle-card");
    }
  },
  methods: {
    generateMonthValue(n) {
      return n < 10 ? `0${n}` : n;
    },
    changeName(e) {
      this.formData.cardName = e.target.value;
      this.$emit("input-card-name", this.formData.cardName);
    },
    changeNumber(e) {
      this.formData.cardNumber = e.target.value;
      let value = this.formData.cardNumber.replace(/\D/g, "");
      // american express, 15 digits
      if (/^3[47]\d{0,13}$/.test(value)) {
        this.formData.cardNumber = value
          .replace(/(\d{4})/, "$1 ")
          .replace(/(\d{4}) (\d{6})/, "$1 $2 ");
        this.cardNumberMaxLength = 17;
      } else if (/^3(?:0[0-5]|[68]\d)\d{0,11}$/.test(value)) {
        // diner's club, 14 digits
        this.formData.cardNumber = value
          .replace(/(\d{4})/, "$1 ")
          .replace(/(\d{4}) (\d{6})/, "$1 $2 ");
        this.cardNumberMaxLength = 16;
      } else if (/^\d{0,16}$/.test(value)) {
        // regular cc number, 16 digits
        this.formData.cardNumber = value
          .replace(/(\d{4})/, "$1 ")
          .replace(/(\d{4}) (\d{4})/, "$1 $2 ")
          .replace(/(\d{4}) (\d{4}) (\d{4})/, "$1 $2 $3 ");
        this.cardNumberMaxLength = 19;
      }
      // eslint-disable-next-line eqeqeq
      if (e.inputType == "deleteContentBackward") {
        let lastChar = this.formData.cardNumber.substring(
          this.formData.cardNumber.length,
          this.formData.cardNumber.length - 1
        );
        // eslint-disable-next-line eqeqeq
        if (lastChar == " ") {
          this.formData.cardNumber = this.formData.cardNumber.substring(
            0,
            this.formData.cardNumber.length - 1
          );
        }
      }
      this.$emit("input-card-number", this.formData.cardNumber);
    },
    changeMonth() {
      this.$emit("input-card-month", this.formData.cardMonth);
    },
    changeYear() {
      this.$emit("input-card-year", this.formData.cardYear);
    },
    changeCvv(e) {
      this.formData.cardCvv = e.target.value;
      this.$emit("input-card-cvv", this.formData.cardCvv);
    },
    invaildCard() {
      let number = this.formData.cardNumber;
      let sum = 0;
      let isOdd = true;
      for (let i = number.length - 1; i >= 0; i--) {
        let num = number.charAt(i);
        if (isOdd) {
          sum += num;
        } else {
          num = num * 2;
          if (num > 9) {
            num = num
              .toString()
              .split("")
              .join("+");
          }
          sum += num;
        }
        isOdd = !isOdd;
      }
      if (sum % 10 !== 0) {
        alert("invaild card number");
      }
    },
    blurCardNumber() {
      if (this.isCardNumberMasked) {
        this.maskCardNumber();
      }
    },
    maskCardNumber() {
      this.mainCardNumber = this.formData.cardNumber;
      let arr = this.formData.cardNumber.split("");
      arr.forEach((element, index) => {
        if (index > 4 && index < 14 && element.trim() !== "") {
          arr[index] = "*";
        }
      });
      this.formData.cardNumber = arr.join("");
    },
    unMaskCardNumber() {
      this.formData.cardNumber = this.mainCardNumber;
    },
    focusCardNumber() {
      this.unMaskCardNumber();
    },
    toggleMask() {
      this.isCardNumberMasked = !this.isCardNumberMasked;
      if (this.isCardNumberMasked) {
        this.maskCardNumber();
      } else {
        this.unMaskCardNumber();
      }
    },
    finishPayment() {
      this.$emit("change-parent");
    }
  }
};
</script>

<style lang="scss">
@import "../assets/scss/card-component.scss";
</style>
