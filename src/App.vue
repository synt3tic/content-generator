<script setup lang="ts">
import {computed, defineAsyncComponent, ref} from "vue";
import type { Component } from "vue";

const tabs = [
    {
        id: 1,
        label: 'Слайдер',
    },
    {
        id: 2,
        label: 'Категории',
    },
    {
        id: 3,
        label: 'Коллекции',
    },
];

const components: Record<number, Component> = {
    1: defineAsyncComponent(() => import('./components/SliderEditor.vue')),
    2: defineAsyncComponent(() => import('./components/CategoriesEditor.vue')),
    3: defineAsyncComponent(() => import('./components/CollectionsEditor.vue')),
};

const selectedTabId = ref(1);
const selectedComponent = computed(() => {
    return components[selectedTabId.value];
})
</script>

<template>
    <h1>Генератор контента для сайта</h1>

    <div class="main">
        <div class="main__tabs">
            <button
                v-for="tab in tabs"
                :key="tab.id"
                :class="{'active-tab': selectedTabId === tab.id}"
                @click="selectedTabId = tab.id"
            >
                {{ tab.label }}
            </button>
        </div>

        <component :is="selectedComponent" />
    </div>
</template>

<style scoped>
.main {
    height: calc(100vh - 64px);
    width: 100%;
}

.active-tab {
    background: #646cff;
    color: white;
}
</style>
