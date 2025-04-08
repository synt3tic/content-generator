<script setup lang="ts">
import {computed, defineAsyncComponent, ref} from "vue";
import type { Component } from "vue";
import type {ContentType} from "./types/main-types.ts";

const tabs: {
    value: ContentType,
    label: string
}[] = [
    {
        value: 'slides',
        label: 'Слайдер',
    },
    {
        value: 'categories',
        label: 'Категории',
    },
    {
        value: 'collections',
        label: 'Коллекции',
    },
];

const components: Record<ContentType, Component> = {
    slides: defineAsyncComponent(() => import('./components/SliderEditor.vue')),
    categories: defineAsyncComponent(() => import('./components/CategoriesEditor.vue')),
    collections: defineAsyncComponent(() => import('./components/CollectionsEditor.vue')),
};

const selectedTabValue = ref<ContentType>(tabs[0].value);
const selectedComponent = computed(() => {
    return components.slides;

    // return components[selectedTabValue.value];
})
</script>

<template>
    <h1 class="title">Генератор контента для сайта</h1>

    <div class="main">
        <div class="main__tabs">
            <button
                v-for="tab in tabs"
                :key="tab.value"
                :class="{'active-tab': selectedTabValue === tab.value}"
                @click="selectedTabValue = tab.value"
            >
                {{ tab.label }}
            </button>
        </div>

        <component :is="selectedComponent" :type="selectedTabValue" />
    </div>
</template>

<style>
.title {
    height: 64px;
    margin: 12px 0 24px;
}

.main {
    height: calc(100vh - 100px);
    width: 100%;
}

.active-tab {
    background: #646cff;
    color: white;
}
</style>
