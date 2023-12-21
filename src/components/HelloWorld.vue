<script setup>
import { ref } from "vue";
import vouchers from "@/contents/code.json";
const codes = Object.keys(vouchers);
console.log(vouchers);

const code = ref("");
let displayCode = ref(null);
let codeError = ref(null);
let claimedPressed = ref(false);

let hasVoucher = () => {
  return displayCode.value != null;
};

let savedCodes = () => {
  let redeemedCodes = JSON.parse(localStorage.getItem('redeemedCodes')) ?? [];
  let redeemed = [];
  codes.forEach((code) => {
    if(redeemedCodes.includes(code))
    {
      redeemed.push(vouchers[code]);
    }
  })
  console.log(redeemed)
  return redeemed
}

function enterCode() {
  let redeemedCodes = JSON.parse(localStorage.getItem('redeemedCodes')) ?? [];
  let value = code.value.toUpperCase();
  if (!codes.includes(value)) {
    codeError.value = "Invlaid code"

    return;
  }

  if (redeemedCodes.includes(value)) {
    codeError.value = "Code Expired"

    return;
  }

  displayCode.value = vouchers[value];
}

function claimCode(event)
{
  let redeemedCodes = JSON.parse(localStorage.getItem('redeemedCodes')) ?? [];
  redeemedCodes.push(code.value.toUpperCase())
  localStorage.setItem('redeemedCodes', JSON.stringify(redeemedCodes))
  claimedPressed.value = true;


  function encode(data) {
      return Object.keys(data)
        .map(
          (key) =>
            encodeURIComponent(key) + "=" + encodeURIComponent(data[key])
        )
        .join("&");
    }

    event.preventDefault();
    fetch("/", {
      method: "POST",
      headers: { "Content-Type": "application/x-www-form-urlencoded" },
      body: encode({
        "form-name": event.target.getAttribute("name"),
        'title': displayCode.value.title
      }),
    })
      .then(() => alert('Thanks for claiming this voucher. I will get in touch with you soon.'))
      .catch((error) => alert(error));
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
          class="text-black rounded py-2 px-3 bg-white border border-2 border-gray-400 outline-none placeholder-gray-600 text-center font-semibold"
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

    <div class=" py-3 px-5 mt-5 text-black">
      <h2 class="font-bold text-xl mb-3 p-2 bg-white rounded shadow-xl">Redeemed</h2>
      <ul>
        <li v-for="(code, index) in savedCodes()" :key="index" class="p-2 bg-white rounded shadow-xl mb-2">
            <span href="" class=" flex items-center justify-center">
              <span>{{ code.title }}</span>
            </span>
        </li>
      </ul>
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
      <form method="POST" data-netlify="true" name="form" @submit.prevent="claimCode($event)">
      <input type="hidden" name="form-name" value="form">
          <input type="hidden" name="data" value="{{JSON.stringify(displayCode)}}">
          <button
          type="submit"
          class="w-full bg-indigo-900 rounded text-white py-2 px-2 shadow font-semibold"
          v-show="!claimedPressed"
        >
          Claim voucher
        </button>
      </form>
    </div>
  </div>
</template>
