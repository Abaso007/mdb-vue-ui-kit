<template>
  <transition
    @before-enter="beforeEnter"
    @enter="enter"
    @after-enter="afterEnter"
    @before-leave="beforeLeave"
    @leave="leave"
    @after-leave="afterLeave"
    enter-active-class="collapsing"
    leave-active-class="collapsing show"
  >
    <component
      v-show="isActive"
      :is="tag"
      :class="className"
      :id="uid"
      ref="collapse"
    >
      <slot />
    </component>
  </transition>
</template>

<script lang="ts">
export default {
  name: "MDBCollapse",
};
</script>

<script setup lang="ts">
import {
  computed,
  ref,
  inject,
  watch,
  watchEffect,
  onMounted,
  provide,
  onUnmounted,
} from "vue";
import type { Ref } from "vue";
import { getUID } from "../../utils/getUID";
import { on, off } from "../../../components/utils/MDBEventHandlers";

const props = defineProps({
  tag: {
    type: String,
    default: "div",
  },
  modelValue: Boolean,
  id: String,
  collapseClass: String,
  duration: {
    type: Number,
    default: 300,
  },
  sidenav: {
    type: Boolean,
    default: false,
  },
  horizontal: {
    type: Boolean,
    default: false,
  },
});
const emit = defineEmits(["update:modelValue"]);

const collapse = ref<HTMLDivElement | HTMLElement | null>(null);
const isActive = ref(props.modelValue);

const className = computed(() => {
  return [
    collapseClass.value,
    props.collapseClass,
    navbarFlexWrapValue && navbarFlexWrapValue.value ? "navbar-collapse" : "",
    showClass.value,
    props.horizontal && "collapse-horizontal",
  ];
});

const collapseClass = computed(() => {
  return props.sidenav
    ? "sidenav-collapse"
    : isActive.value
    ? "collapse"
    : null;
});

const accordionState = inject<{
  itemsCount: number;
  active: null;
} | null>("accordionState", null);
const incrementAccordionItemsCount = inject<(() => number) | false>(
  "incrementAccordionItemsCount",
  false
);
const setAccordionActiveItem = inject<((element: number) => void) | false>(
  "setAccordionActiveItem",
  false
);
const index = ref<number | null>(null);

const manageAccordion = () => {
  if (index.value !== null && isActive.value) {
    setAccordionActiveItem && setAccordionActiveItem(index.value);
  }
};

watchEffect(
  () => {
    if (accordionState) {
      if (accordionState.active !== index.value) {
        emit("update:modelValue", false);
      }
    }
  },
  { flush: "post" }
);

onMounted(() => {
  if (isActive.value && collapse.value) {
    collapse.value.style.height = collapse.value.scrollHeight + "px";
    collapse.value.style.width = collapse.value.scrollWidth + "px";
  }

  if (accordionState) {
    index.value = incrementAccordionItemsCount
      ? incrementAccordionItemsCount()
      : 0;

    if (isActive.value) {
      setAccordionActiveItem && setAccordionActiveItem(index.value);
    }
  }
});

let isCollapsing = false;
const emitInterval = ref(0);

watch(
  () => props.modelValue,
  (cur) => {
    if (isCollapsing) {
      clearInterval(emitInterval.value);
      emitInterval.value = setTimeout(() => {
        emit("update:modelValue", isActive.value);
      }, props.duration);
      return;
    }

    isActive.value = cur;

    if (accordionState) {
      manageAccordion();
    }
  }
);

const openCollapse = () => {
  emit("update:modelValue", true);
};

provide("openCollapse", openCollapse);

const navbarFlexWrapValue = inject<Ref<string> | false>(
  "navbarFlexWrapValue",
  false
);

const showClass = computed(() => {
  if (
    !navbarFlexWrapValue ||
    (navbarFlexWrapValue.value === "wrap" && isActive.value)
  ) {
    return "show";
  } else if (navbarFlexWrapValue.value === "nowrap" && isActive.value) {
    return false;
  }

  return false;
});

const checkWrapCollapseValue = (
  _cur: string,
  prev: string | boolean | undefined
) => {
  if (prev === "null" && props.modelValue) {
    // open on first render when collapsed props
    isActive.value = true;
  } else if (prev === "null" && !props.modelValue) {
    // close on first render when no collapsed props
    isActive.value = false;
  } else if (prev === "nowrap") {
    // always close when resizing down from full navbar
    isActive.value = false;
  }
};

watch(
  () => navbarFlexWrapValue && navbarFlexWrapValue.value,
  (cur, prev) => {
    if (cur === "nowrap") {
      isActive.value = true;
    } else if (cur === "wrap") {
      checkWrapCollapseValue(cur, prev);
    }
    emit("update:modelValue", isActive.value);
  },
  { immediate: true }
);

const uid = computed(() => {
  return props.id ? props.id : getUID("collapsibleContent-");
});

const beforeEnter = (el: Element) => {
  const element = el as HTMLElement;
  if (props.horizontal) {
    element.style.width = "0";
  } else {
    element.style.height = "0";
  }
  element.style.transitionDuration = props.duration + "ms";
  isCollapsing = true;
};
const enter = (el: Element) => {
  const element = el as HTMLElement;
  if (props.horizontal) {
    element.style.width = collapse.value?.scrollWidth + "px";
  } else {
    element.style.height = collapse.value?.scrollHeight + "px";
  }
};

const afterEnter = (el: Element) => {
  const element = el as HTMLElement;
  if (!element.classList.contains("show")) {
    element.classList.add("show");
  }

  if (props.horizontal) {
    element.style.width = "";
  } else {
    element.style.height = "";
  }

  isCollapsing = false;
};

const beforeLeave = (el: Element) => {
  const element = el as HTMLElement;
  if (props.horizontal) {
    if (!element.style.width) {
      element.classList.add("show");
      element.style.width = collapse.value?.scrollWidth + "px";
    }
  } else {
    if (!element.style.height) {
      element.classList.add("show");
      element.style.height = collapse.value?.scrollHeight + "px";
    }
  }
  isCollapsing = true;
};
const leave = (el: Element) => {
  const element = el as HTMLElement;
  if (props.horizontal) {
    element.style.width = "0px";
  } else {
    element.style.height = "0px";
  }
};

const afterLeave = (el: Element) => {
  const element = el as HTMLElement;
  element.classList.add("collapse");
  isCollapsing = false;
};

const previousWindowWidth = ref<number | null>(null);
const isThrottled = ref(false);

const handleResize = () => {
  if (!isActive.value || isThrottled.value) return;

  isThrottled.value = true;

  const windowWidth = window.innerWidth;
  previousWindowWidth.value = windowWidth;
  setTimeout(() => {
    isThrottled.value = false;
  }, 100);
};

onMounted(() => {
  previousWindowWidth.value = window.innerWidth;
  on(window, "resize", handleResize);
});

onUnmounted(() => {
  off(window, "resize", handleResize);
});

defineExpose({ collapse });
</script>
