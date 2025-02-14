<template>
  <div>
    <div ref="referenceElement" @click="toggleDropdown">
      <slot name="dropdown-button" />
    </div>
    <div
      v-show="visible"
      ref="popperElement"
      :class="`popper-container z-50 ${
        snapLeft ? 'popper-container-snap-left' : ''
      }`"
    >
      <transition
        name="dropdown"
        @before-enter="afterEnter"
        @after-leave="afterLeave"
      >
        <div v-if="showDropdown" class="h-full">
          <slot name="dropdown-content" />
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, getCurrentInstance } from "vue";

// prettier-ignore
import { createPopper, Instance, Modifier, OptionsGeneric } from "@popperjs/core"; // eslint-disable-line import/named

const showDropdown = ref<boolean>(false);
const snapLeft = ref(false);
const visible = ref(false);
const referenceElement = ref<HTMLElement | null>(null);
const popperElement = ref<HTMLElement | null>(null);

const windowLocked = useState("windowLocked", () => false);

let popperInstance: Instance | null = null;

const toggleDropdown = () => {
  showDropdown.value = !showDropdown.value;
  if (!showDropdown.value) {
    windowLocked.value = false;
  }
};

const closeDropdown = () => {
  showDropdown.value = false;
  windowLocked.value = false;
};

function destroyPopper() {
  // Destroy the existing Popper.js instance if it exists
  if (popperInstance) {
    popperInstance.destroy();
    popperInstance = null;
  }
}
function isHTMLElement(value: unknown): value is HTMLElement {
  return value instanceof HTMLElement;
}

const createPopperInstance = (
  newReferenceElement: HTMLElement,
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  options: Partial<OptionsGeneric<Partial<Modifier<any, any>>>>
) => {
  if (!newReferenceElement) {
    return;
  }

  // Create a new Popper.js instance with the new reference element
  if (isHTMLElement(popperElement.value)) {
    popperInstance = createPopper(
      newReferenceElement,
      popperElement.value,
      options
    );
    popperInstance.forceUpdate();
  }
};

const updatePlacement = async () => {
  const viewportWidth = window.innerWidth;
  const minWidthBreakpoint = 768; // md

  if (showDropdown.value) {
    if (viewportWidth < minWidthBreakpoint) {
      destroyPopper();
      windowLocked.value = true;
      snapLeft.value = true;
    } else if (
      viewportWidth >= minWidthBreakpoint &&
      referenceElement.value &&
      !popperInstance
    ) {
      windowLocked.value = false;
      snapLeft.value = false;
      await createPopperInstance(referenceElement.value, {
        placement: "bottom",
        strategy: "fixed",
        modifiers: [
          {
            name: "offset",
            options: {
              offset: [0, 14],
            },
          },
        ],
      });
    }
  }
};

watch([showDropdown, referenceElement], async () => {
  if (showDropdown.value && referenceElement.value) {
    await updatePlacement();
  }
});

const handleClick = (event: MouseEvent) => {
  if (
    referenceElement.value &&
    popperElement.value &&
    showDropdown.value &&
    !referenceElement.value.contains(event.target as Node) &&
    !popperElement.value.contains(event.target as Node)
  ) {
    event.preventDefault();
    event.stopPropagation();
    closeDropdown();
  } else if (
    popperElement.value &&
    popperElement.value.contains(event.target as Node)
  ) {
    // close dropdown automatically if we click on an internal link inside
    const targetElement = event.target as HTMLElement;
    if (targetElement.nodeName === "A") {
      const hrefAttribute = targetElement.getAttribute("href");
      if (
        hrefAttribute &&
        (hrefAttribute.startsWith("/") || hrefAttribute.startsWith("#"))
      ) {
        closeDropdown();
      }
    }
  }
};

onMounted(() => {
  // Attach dropdown content to body
  if (popperElement.value) {
    document.body.appendChild(popperElement.value);
  }

  const instance = getCurrentInstance();
  if (instance) {
    instance.emit("register-toggle", toggleDropdown);
  }

  // Handle click outside
  document.addEventListener("click", handleClick);

  // Handle window resize
  window.addEventListener("resize", updatePlacement);
});

onUnmounted(() => {
  document.removeEventListener("click", handleClick);
  window.removeEventListener("resize", updatePlacement);

  // Detach dropdown content to body
  if (popperElement.value) {
    popperElement.value.remove();
  }

  destroyPopper();
});

const afterLeave = () => {
  visible.value = false;
  destroyPopper();
};
const afterEnter = () => {
  visible.value = true;
};

provide("toggleDropdown", toggleDropdown);
</script>

<style lang="postcss">
@media (max-width: 767px) {
  .dropdown-container,
  .dropdown-container::before {
    @apply ltr:!rounded-r-3xl rtl:!rounded-l-3xl;
  }
}
</style>
<style lang="postcss" scoped>
/** Custom default dropdown container */
.popper-container:deep(.dropdown-container) {
  @apply h-full will-change-transform z-50 rounded-lg text-white/90 relative list-none backdrop-blur-xl backdrop-brightness-75 md:bg-[#042248] border-black border border-opacity-30 shadow-[0_10px_15px_-3px_rgba(0,0,0,.1),0_4px_6px_-4px_rgba(0,0,0,.1),0px_0px_0px_1px_rgba(255,255,255,.05)_inset];
}

@keyframes animate {
  0% {
    @apply opacity-0;
  }
  50% {
    @apply opacity-100;
  }
  100% {
    @apply opacity-0;
  }
}

.popper-container-snap-left {
  @apply h-screen fixed top-0 ltr:left-0 rtl:right-0 overscroll-y-contain;
  &:deep(.dropdown-container) {
    @apply !w-[49vw] sm:!w-[45vw] shadow-beam-pink shadow-2xl;
  }
  &:deep(.dropdown-container)::before {
    @apply shadow-beam-green shadow-2xl absolute w-full h-full -z-[1];
    animation: animate 15s linear infinite;
    content: "";
  }
}
.dropdown-enter-active,
.dropdown-leave-active {
  transition: opacity 225ms ease-in-out, transform 225ms ease-in-out;
}

@media (max-width: 767px) {
  .dropdown-enter-from,
  .dropdown-leave-to {
    @apply ltr:-translate-x-full rtl:translate-x-full;
  }
  .dropdown-enter-to,
  .dropdown-leave-from {
    @apply ltr:-translate-x-0 rtl:translate-x-0;
  }
}

@media (min-width: 768px) {
  .dropdown-enter-from,
  .dropdown-leave-to {
    @apply opacity-0 ltr:!scale-95 rtl:!scale-95;
  }

  .dropdown-enter-to,
  .dropdown-leave-from {
    @apply opacity-100 ltr:!scale-100 rtl:!scale-100;
  }
}
</style>
