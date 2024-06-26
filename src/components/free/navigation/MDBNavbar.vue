<template>
  <component :is="tag" ref="navbar" :class="navClass" role="navigation">
    <div v-if="container" :class="containerClass">
      <slot />
    </div>
    <slot v-if="!container" />
  </component>
</template>

<script lang="ts">
export default {
  name: "MDBNavbar",
};
</script>

<script setup lang="ts">
import { computed, ref, provide, onMounted } from "vue";

const props = defineProps({
  tag: {
    type: String,
    default: "nav",
  },
  bg: String,
  dark: {
    type: Boolean,
    default: false,
  },
  light: {
    type: Boolean,
    default: false,
  },
  double: {
    type: Boolean,
    default: false,
  },
  expand: String,
  position: String,
  transparent: {
    type: Boolean,
    default: false,
  },
  scrolling: {
    type: Boolean,
    default: false,
  },
  scrollingOffset: {
    type: Number,
    default: 100,
  },
  center: {
    type: Boolean,
    default: false,
  },
  container: {
    type: [Boolean, String],
    default: false,
  },
  classContainer: String,
  classNavbar: String,
});

const navClass = computed(() => {
  return [
    "navbar",
    props.dark && "navbar-dark",
    props.light && "navbar-light",
    props.bg && !props.transparent ? `bg-${props.bg}` : "",
    props.expand
      ? props.expand === "small" || props.expand === "sm"
        ? "navbar-expand-sm"
        : props.expand === "medium" || props.expand === "md"
        ? "navbar-expand-md"
        : props.expand === "large" || props.expand === "lg"
        ? "navbar-expand-lg"
        : "navbar-expand-xl"
      : "",
    props.position === "top"
      ? "fixed-top"
      : props.position === "bottom"
      ? "fixed-bottom"
      : props.position === "sticky"
      ? "sticky-top"
      : "",
    props.scrolling && scrollingClass.value,
    props.double && "double-nav",
    props.center && "justify-content-center",
    props.classNavbar,
  ];
});

const containerClass = computed(() => {
  if (!props.container) {
    return false;
  }
  return [
    props.container !== true
      ? `container-${props.container}`
      : "container-fluid",
    props.classContainer && props.classContainer,
  ];
});

const scrollingClass = ref("navbar-scroll");

const handleScroll = () => {
  if (window.pageYOffset > props.scrollingOffset) {
    scrollingClass.value = "navbar-scroll navbar-scrolled";
  } else {
    scrollingClass.value = "navbar-scroll";
  }
};

const navbar = ref<HTMLElement | null>(null);
const navbarFlexWrapValue = ref("nowrap");
provide("navbarFlexWrapValue", navbarFlexWrapValue);

const handleResize = () => {
  if (!navbar.value) return;

  const wrap = getComputedStyle(navbar.value).flexWrap;

  if (wrap === "nowrap") {
    navbarFlexWrapValue.value = "nowrap";
  } else if (wrap === "wrap") {
    navbarFlexWrapValue.value = "wrap";
  }
};

onMounted(() => {
  if (
    getComputedStyle(navbar.value as HTMLElement) &&
    getComputedStyle(navbar.value as HTMLElement).flexWrap === "wrap"
  ) {
    navbarFlexWrapValue.value = "wrap";
  } else {
    navbarFlexWrapValue.value = "nowrap";
  }
  window.addEventListener("resize", () => handleResize());

  if (props.scrolling) {
    window.addEventListener("scroll", handleScroll);
  }
  return () => {
    window.removeEventListener("scroll", handleScroll);
  };
});
</script>
