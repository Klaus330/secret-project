<script setup>
import { ref } from "vue";
import vouchers from "@/contents/code.json";
const codes = Object.keys(vouchers);
console.log(codes);

const code = ref("");
let displayCode = ref(null);
let codeError = ref(null);
let claimedPressed = ref(false);

let hasVoucher = () => {
  return displayCode.value != null;
};

function enterCode() {
  let value = code.value.toUpperCase();
  if (!codes.includes(value)) {
    codeError.value = "Invlaid code"

    return;
  }

  let redeemedCodes = localStorage.getItem('redeemedCodes') ?? [];
  if (redeemedCodes.includes(value)) {
    codeError.value = "Code Expired"

    return;
  }

  displayCode.value = vouchers[value];
}

function claimCode()
{
  let redeemedCodes = localStorage.getItem('redeemedCodes') ?? [];
  redeemedCodes.push(code.value.toUpperCase())
  localStorage.setItem('redeemedCodes', JSON.stringify(redeemedCodes))
  claimedPressed.value = true;
  alert('Thanks for claiming this voucher. I will get in touch with you soon.')
}
</script>

<template>
  <div id="" v-if="!hasVoucher()">
    <div class="pb-5">
      <h2 class="text-white font-bold text-5xl drop-shadow-xl uppercase">Vouchers</h2>
    </div>
    <div
      class="flex items-center justify-between flex-col bg-white rounded px-4 py-6 gap-6 shadow-xl shadow-purple-600"
    >
      <div>
        <input
          type="text"
          name="code"
          v-model="code"
          class="rounded py-2 px-3 bg-white border border-2 border-gray-400 outline-none placeholder-gray-600 text-center font-semibold"
          :class="{'border-red-500 placeholder-red-500': codeError}"
          placeholder="Enter CODE"
        />
        <p class="text-red-500 mt-1" id="code-error" v-show="codeError != null">{{ codeError }}</p>
      </div>
      <button
        type="button"
        class="w-full bg-gray-900 rounded text-white py-2 px-2 shadow font-semibold"
        @click="enterCode()"
      >
        Enter
      </button>
    </div>
  </div>

  <div
    class="bg-black bg-opacity-20 text-white p-5 mt-10 rounded"
    v-else
  >

    <div class="p-3 rounded bg-white mt-5 text-black shadow-xl">
      <div class="border border-black rounded">
        <div class="p-3 border-b border-black">
          <p>
            {{ displayCode.emoji }}
          </p>
          <h3 class="font-bold text-xl lg:text-5xl text-red-900 drop-shadow">
            {{ displayCode.title }}
          </h3>
          <p>
            {{ displayCode.emoji }}
          </p>
        </div>
        <div class="p-3">
          {{ displayCode.description }}
        </div>
        <div class="border-t border-black bg-red-100 p-1 text-md">
          {{ displayCode.short_description }}
        </div>
      </div>
    </div>

    <div class="mt-5">
      <form method="POST" data-netlify="true">
      <input type="hidden" name="form-name" value="form">
          <input type="hidden" name="data" value="{{JSON.stringify(displayCode)}}">
          <button
          type="button"
          class="w-full bg-indigo-900 rounded text-white py-2 px-2 shadow font-semibold"
          @click="claimCode()"
          v-show="!claimedPressed"
        >
          Claim voucher
        </button>
      </form>
    </div>
  </div>
</template>
