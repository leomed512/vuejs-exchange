<template>
  <header class="shadow w-screen">
    <nav>
      <nav class="flex items-center justify-between flex-wrap bg-green-700 p-6">
        <div class="flex items-center flex-shrink-0 text-white mr-6">
          <px-icon class="mr-2" />
          <router-link
            :to="{ name: 'home' }"
            class="font-semibold text-xl tracking-tight hover:underline"
            >Mérida Exchange
          </router-link>
        </div>
        <div class="hidden flex-grow sm:flex sm:items-center lg:w-auto">
          <div class="text-sm lg:flex-grow">
            <router-link
              v-for="l in links"
              :key="l.title"
              :to="l.to"
              class="block sm:inline-block lg:mt-0 text-teal-200 hover:text-white mr-4"
              >{{ l.title }}</router-link
            >
          </div>
        </div>
      </nav>
    </nav>
  </header>
</template>

<script>
import PxIcon from "@/components/PxIcon.vue";
export default {
  name: "PxHeader",
  components: { PxIcon },
  props: {
    links: {
      type: Array,
      default: () => [],
    },
  },
};
</script>
