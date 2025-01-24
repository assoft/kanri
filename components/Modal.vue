<template>
    <transition name="modal-fade">
        <div
            @click.self="clickOutsideToClose ? $emit('closeModal') : () => {}"
            class="modal inset-0 z-50 flex h-screen w-screen flex-col items-center justify-center bg-zinc-800 bg-opacity-40 bg-clip-padding backdrop-filter"
            :class="blurBackground ? 'backdrop-blur-xl' : 'backdrop-brightness-50'"
        >
            <div
                class="bg-elevation-1 min-h-content min-w-content rounded-md py-4 pl-8 pr-6 shadow-lg"
            >
                <slot name="content" class="p-4"></slot>
            </div>
        </div>
    </transition>
</template>

<script setup lang="ts">
withDefaults(
    defineProps<{
        clickOutsideToClose?: boolean;
        blurBackground?: boolean;
    }>(),
    {
        clickOutsideToClose: true,
        blurBackground: true,
    }
);

const emit = defineEmits(['closeModal']);

onMounted(() => {
    document.addEventListener("keydown", keyDownListener);
});

onBeforeUnmount(() => {
    document.removeEventListener("keydown", keyDownListener);
});

const keyDownListener = (e: { key: string; }) => {
    if (e.key === "Escape") {
        emit("closeModal");
    }
};
</script>

<style>
.modal-fade-enter-active,
.modal-fade-leave-active {
    transition: opacity 0.35s ease-out;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
    opacity: 0;
}

.modal {
    position: fixed;
    max-width: 100%;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
}
</style>
