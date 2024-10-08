<script>
import SEO from "$components/SEO.svelte"
import Countdown from "svelte-countdown"
import { fade, slide } from "svelte/transition"

let { data } = $props()
let currentDate = $state(new Date().toISOString())
$effect(() => {
  const interval = setInterval(() => {
    currentDate = new Date().toISOString()
  }, 1000)
  return () => clearInterval(interval)
})

const products = data.products?.data
// const discounts = data.discounts?.data

const fetchLimitedTimeDiscount = (async () => {
  const response = await fetch("https://api.daisyui.com/api/discount.json")
  return await response.json()
})()

const publishedProducts = products.filter((product) => {
  return product.attributes.status === "published"
})
const indevelopmentProducts = products.filter((product) => {
  return product.attributes.status === "draft"
})
function convertCurrency(number) {
  const formatted = (number / 100).toFixed(2)
  return `$${formatted.endsWith(".00") ? formatted.slice(0, -3) : formatted}`
}
function extractUUID(url) {
  const regex = /\/buy\/([a-f0-9-]{36})(?:\?|$)/
  const match = url.match(regex)
  return match ? match[1] : null
}
const dateFormat = {
  year: "numeric",
  month: "numeric",
  day: "numeric",
  hour: "2-digit",
  minute: "2-digit",
  second: "2-digit",
}

let isClipboardButtonPressed = $state(false)
const copyText = (text) => {
  navigator.clipboard.writeText(text)
  isClipboardButtonPressed = true
  setTimeout(() => {
    isClipboardButtonPressed = false
  }, 2000)
}
</script>

<SEO title="Official daisyUI Store" desc="daisyUI Store - Professional templates made by daisyUI" />
{#await fetchLimitedTimeDiscount}

{:then discount}
  {#if discount.data.attributes.expires_at && new Date(discount.data.attributes.expires_at).toISOString() > currentDate}
    <div class="alert h-24 my-10" transition:slide={{ duration: 400 }}>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="text-base-content/50 mx-2 h-5 w-5 shrink-0 stroke-current max-lg:hidden">
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M9.568 3H5.25A2.25 2.25 0 003 5.25v4.318c0 .597.237 1.17.659 1.591l9.581 9.581c.699.699 1.78.872 2.607.33a18.095 18.095 0 005.223-5.223c.542-.827.369-1.908-.33-2.607L11.16 3.66A2.25 2.25 0 009.568 3z">
        </path>
        <path stroke-linecap="round" stroke-linejoin="round" d="M6 6h.008v.008H6V6z"></path>
      </svg>
      <div class="flex w-full flex-col items-center justify-between gap-10 sm:flex-row" transition:fade={{ duration: 400 }}>
        <div class="flex flex-col gap-1">
          <h2 class="text-lg font-bold">
            {discount.data.attributes.name}
          </h2>
          <div class="text-base-content/60 text-sm [text-wrap:balance]">
            Use <span
              data-tip="{isClipboardButtonPressed ? 'copied' : 'copy'}"
              class="tooltip badge badge-outline">
              <button
                class="font-mono tracking-wide"
                onclick="{() => copyText(discount.data.attributes.code)}">
                {discount.data.attributes.code}
              </button>
            </span>
            code at checkout to get {discount.data.attributes.amount}% discount on all products.
          </div>
        </div>

        {#if discount.data.attributes.expires_at}
          <Countdown
            from="{new Date(discount.data.attributes.expires_at).toLocaleString('en-GB', dateFormat)}"
            dateFormat="DD/MM/YYYY, HH:mm:ss"
            let:remaining>
            {#if remaining.done === false}
              <div class="tooltip shrink-0 after:hidden" data-tip="Remaining time" transition:fade={{ duration: 400 }}>
                <date
                  datetime="{new Date(discount.data.attributes.expires_at).toLocaleString(
                    'en-GB',
                    dateFormat
                  )}"
                  class="grid grid-cols-3 gap-2 text-center font-mono text-xs">
                  <!-- <div class="border-base-content/20 rounded-btn border border-dashed p-2">
                    <span class="countdown block text-xl">
                      <span style="{`--value:${remaining.days};`}"></span>
                    </span>
                    <span class="text-base-content/40 text-xs">day</span>
                  </div> -->
                  <div class="border-base-content/20 rounded-btn border border-dashed p-2">
                    <span class="countdown block text-xl">
                      <span style="{`--value:${remaining.hours};`}"></span>
                    </span>
                    <span class="text-base-content/40 text-xs">hour</span>
                  </div>
                  <div class="border-base-content/20 rounded-btn border border-dashed p-2">
                    <span class="countdown block text-xl">
                      <span style="{`--value:${remaining.minutes};`}"></span>
                    </span>
                    <span class="text-base-content/40 text-xs">min</span>
                  </div>
                  <div class="border-base-content/20 rounded-btn border border-dashed p-2">
                    <span class="countdown block text-xl">
                      <span style="{`--value:${remaining.seconds};`}"></span>
                    </span>
                    <span class="text-base-content/40 text-xs">sec</span>
                  </div>
                </date>
              </div>
            {:else if !data}
              <div class="border-base-content/20 rounded-btn shrink-0 border border-dashed p-2">
                Ended
              </div>
            {/if}
          </Countdown>
        {/if}
      </div>
    </div>
  {/if}
{/await}


<!-- published -->
<div class="mx-auto flex max-w-7xl flex-col gap-16">
  {#each publishedProducts as product}
    <div
      class="rounded-box relative grid grid-cols-12 gap-y-10 py-10 xl:gap-x-10"
      id="{product.id}">
      <div class="col-span-12 row-start-2 flex flex-col gap-8 xl:col-span-5 xl:row-start-1">
        <div>
          {#if product.customattributes?.tags}
            <span class="flex gap-2">
              {#each product.customattributes.tags as tag}
                <span class="badge badge-success badge-outline italic">{tag}</span>
              {/each}
            </span>
          {/if}
          <h2 class="font-title text-lg font-black [text-wrap:balance] sm:text-3xl xl:text-5xl">
            {product.attributes.name}
          </h2>
        </div>
        <div class="flex items-start justify-between">
          <div class="flex gap-2">
            {#if product.customattributes?.originalprice}
              <span class="text-2xl line-through opacity-40 xl:text-4xl">
                &nbsp;{convertCurrency(product.customattributes?.originalprice)}&nbsp;
              </span>
            {/if}
            <span class="flex flex-col">
              <span class="flex items-center gap-2">
                {#if product.customattributes?.displayprice}
                  <span class="font-title text-2xl font-light xl:text-5xl">
                    {convertCurrency(product.customattributes?.displayprice)}
                  </span>
                {:else if product.attributes.from_price && product.attributes.to_price && product.attributes.from_price !== product.attributes.to_price}
                  From
                  <span class="font-title text-2xl font-light xl:text-5xl">
                    {convertCurrency(product.attributes.from_price)}
                  </span>
                  to
                  <span class="font-title text-2xl font-light xl:text-5xl">
                    {convertCurrency(product.attributes.to_price)}
                  </span>
                {:else}
                  <span class="font-title text-2xl font-light xl:text-5xl">
                    {convertCurrency(product.attributes.price)}
                  </span>
                {/if}
              </span>

              {#if product.customattributes?.displaypricenote}
                <span class="text-sm italic opacity-40">
                  {product.customattributes?.displaypricenote}
                </span>
              {/if}
            </span>
          </div>
          <div class="flex flex-col items-center gap-3">
            <a
              href="{product.customattributes.ref ? `/store/checkout?product=${extractUUID(product.attributes.buy_now_url)}&aff=${product.customattributes.ref}` : product.attributes.buy_now_url}"
              class="btn btn-primary shadow-primary/50 group shrink-0 rounded-full shadow xl:px-10"
              target="_blank"
              rel="noopener noreferrer">
              Get it now
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke-width="1.5"
                stroke="currentColor"
                class="hidden h-6 w-6 transition-transform duration-300 group-hover:translate-x-1 rtl:rotate-180 rtl:group-hover:-translate-x-1 md:inline-block">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M17.25 8.25L21 12m0 0l-3.75 3.75M21 12H3">
                </path>
              </svg>
            </a>
            {#if product.customattributes?.preveiw_url}
              <a class="link text-xs" href="{product.customattributes.preveiw_url}" target="_blank">
                Live Preview
              </a>
            {/if}
          </div>
        </div>
        {#if product.attributes.description}
          <div
            class="prose prose-sm prose-li:my-0 prose-ul:leading-none prose-li:leading-normal prose-p:my-2 prose-ul:my-2 text-xs [text-wrap:balance]">
            {@html product.attributes.description}
          </div>
        {/if}
      </div>
      <div class="col-span-12 row-start-1 flex flex-col gap-6 xl:col-span-7">
        <a
          target="_blank"
          href="{product.customattributes?.screenshot
            ? product.customattributes?.screenshot
            : product.attributes.large_thumb_url}"
          rel="noopener noreferrer"
          class="rounded-box border-base-200 group relative block aspect-[4/3] overflow-hidden border object-cover">
          <div
            class="absolute inset-0 z-[1] grid place-content-center bg-black/50 opacity-0 transition-all duration-500 group-hover:scale-150 group-hover:opacity-100">
            <svg
              class="text-white"
              width="32"
              height="32"
              viewBox="0 0 48 48"
              fill="none"
              xmlns="http://www.w3.org/2000/svg">
              <path
                d="M21 38C30.3888 38 38 30.3888 38 21C38 11.6112 30.3888 4 21 4C11.6112 4 4 11.6112 4 21C4 30.3888 11.6112 38 21 38Z"
                fill="none"
                stroke="currentColor"
                stroke-width="4"
                stroke-linejoin="round">
              </path>
              <path
                d="M21 15L21 27"
                stroke="currentColor"
                stroke-width="4"
                stroke-linecap="round"
                stroke-linejoin="round">
              </path>
              <path
                d="M15.0156 21.0156L27 21"
                stroke="currentColor"
                stroke-width="4"
                stroke-linecap="round"
                stroke-linejoin="round">
              </path>
              <path
                d="M33.2216 33.2217L41.7069 41.707"
                stroke="currentColor"
                stroke-width="4"
                stroke-linecap="round"
                stroke-linejoin="round">
              </path>
            </svg>
          </div>
          <img
            style="{`background-image: url('${product.attributes.thumb_url}')`}"
            src="{product.attributes.large_thumb_url}"
            alt="{product.attributes.name}"
            loading="lazy"
            class="bg-base-300 aspect-[4/3] w-full bg-cover bg-center object-cover" />
        </a>
        {#if product.customattributes?.tech}
          <div class="flex items-center justify-center gap-4 md:justify-end">
            <span class="text-base-content/50 text-xs italic">made with</span>
            {#each product.customattributes.tech as tech}
              <div
                class="lg:border-base-content/10 tooltip grid place-content-center rounded-full border-dashed lg:border lg:p-2 xl:p-3"
                data-tip="{data.tech[tech]}">
                <img
                  class="aspect-square w-5 xl:w-6"
                  src="{`https://img.daisyui.com/images/logos/${tech}.svg`}"
                  alt="{tech}" />
              </div>
            {/each}
          </div>
        {/if}
      </div>
    </div>
    {#if product !== products[products.length - 1]}
      <!-- <div class="divider before:bg-base-content/5 after:bg-base-content/5"></div> -->
    {/if}
  {:else}
    <div class="lg:col-span-3 flex justify-center items-center font-bold text-base-content/20">
      Coming soon…
    </div>
  {/each}
</div>

<!-- coming soon -->
<div class="divider text-base-content/30 my-20">In development</div>
<div class="grid gap-12 lg:grid-cols-3">
  {#each indevelopmentProducts.slice(0, 3) as product}
    <div
      class="rounded-box border-base-300 text-base-content/30 flex h-72 flex-col items-center justify-center gap-6 border-2 border-dashed p-10 text-center [text-wrap:balance]">
      <svg
        width="16"
        height="16"
        viewBox="0 0 48 48"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <rect
          x="4"
          y="10"
          width="40"
          height="30"
          rx="2"
          fill="none"
          stroke="currentColor"
          stroke-width="4"
          stroke-linecap="round"
          stroke-linejoin="round">
        </rect>
        <path d="M14 6V14" stroke="currentColor" stroke-width="4" stroke-linecap="round"></path>
        <path d="M25 23L14 23" stroke="currentColor" stroke-width="4" stroke-linecap="round"></path>
        <path d="M34 31L14 31" stroke="currentColor" stroke-width="4" stroke-linecap="round"></path>
        <path d="M34 6V14" stroke="currentColor" stroke-width="4" stroke-linecap="round"></path>
      </svg>
      <div>{product.attributes.name}</div>
    </div>
  {:else}
    <div class="lg:col-span-3 flex justify-center items-center font-bold text-base-content/20">
      Coming soon…
    </div>
  {/each}
</div>

<div class="divider text-base-content/30 my-20"></div>

<div id="mc_embed_shell" class="card bg-base-200 mt-10">
  <div class="card-body flex flex-col gap-4">
    <h2 class="text-xl font-black lg:text-4xl">
      Get notified about upcoming products and discounts!
    </h2>
    <div>
      <p class="font-bold">
        Subscribe to daisyUI store newsletter to get updates about new products and discounts codes.
      </p>
      <p class="text-base-content/60 text-xs">
        You will only receive a single email when a new product is added or when a new discount code
        is available. No spam. No ads.
      </p>
    </div>
    <div id="mc_embed_signup">
      <form
        action="https://gmail.us8.list-manage.com/subscribe/post?u=42813cff910e47b1bd132369a&amp;id=9fd7333f07&amp;f_id=003d03e0f0"
        method="post"
        id="mc-embedded-subscribe-form"
        name="mc-embedded-subscribe-form"
        target="_blank"
        novalidate="">
        <!-- store group -->
        <input type="checkbox" name="group[347002][8]" value="1" class="hidden" checked="checked" />
        <!-- discounts group -->
        <input type="checkbox" name="group[347002][1]" value="1" class="hidden" checked="checked" />
        <div class="form-control w-full max-w-sm">
          <label class="label" for="mce-EMAIL">
            <span class="label-text">Email Address</span>
          </label>
          <div class="join">
            <input
              type="email"
              name="EMAIL"
              class="join-item input input-bordered w-full max-w-sm"
              id="mce-EMAIL"
              placeholder="mail@site.com"
              required />
            <button name="subscribe" class="join-item btn btn-success">Subscribe</button>
          </div>
        </div>
        <div aria-hidden="true" class="hidden">
          <input type="text" name="b_42813cff910e47b1bd132369a_9fd7333f07" tabindex="-1" value="" />
        </div>
      </form>
    </div>
  </div>
</div>
