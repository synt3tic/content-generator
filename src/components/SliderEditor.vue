<script setup lang="ts">
import {computed, onMounted, ref} from "vue";
import type {Banner, ContentType} from "../types/main-types.ts";

const props = defineProps<{
    type: ContentType;
}>();

const DEFAULT_VALUES = [
    {
        id: 1,
        link: '',
        text: '',
        img: '',
    },
    {
        id: 2,
        link: '',
        text: '',
        img: '',
    }
];

const slides = ref<Record<ContentType, Banner[]>>({
    slides: structuredClone(DEFAULT_VALUES),
    categories: structuredClone(DEFAULT_VALUES),
    collections: structuredClone(DEFAULT_VALUES),
});

const createEmptySlide = () => {
    slides.value[props.type].push({
        id: slides.value[props.type][slides.value[props.type].length - 1].id + 1,
        link: '',
        text: '',
        img: '',
    });
}

const removeSlide = (id: number) => {
    slides.value[props.type] = slides.value[props.type].filter(slide => slide.id !== id);
};

const copyToClipboard = () => {
    navigator.clipboard.writeText(preview.value)
        .then(() => {
            alert('Скопировано в буфер обмена!');
        })
        .catch(err => {
            console.error('Ошибка копирования: ', err);
        });
};

const preview = computed(() => {
    const string = JSON.stringify(slides.value[props.type].map(({link, text, img}) => ({link, text, img})));

   return `const ${props.type} = ${string}`;
});

const saveResultToLocalStorage = () => {
    localStorage.setItem(`result_${props.type}`, JSON.stringify(slides.value[props.type]));
    alert('Сохранено')
}

const reset = () => {
    localStorage.removeItem(`result_${props.type}`);
    slides.value[props.type] = DEFAULT_VALUES;
    alert('Очищено')
}

const resetAll = () => {
    localStorage.removeItem('result_slides');
    localStorage.removeItem('result_collections');
    localStorage.removeItem('result_categories');
    slides.value.slides = structuredClone(DEFAULT_VALUES);
    slides.value.collections = structuredClone(DEFAULT_VALUES);
    slides.value.categories = structuredClone(DEFAULT_VALUES);
    alert('Очищено всё')
}

onMounted(() => {
    const slidesJSON = localStorage.getItem('result_slides');
    const categoriesJSON = localStorage.getItem('result_categories');
    const collectionsJSON = localStorage.getItem('result_collections');

    if (slidesJSON) {
        slides.value.slides = JSON.parse(slidesJSON);
    }
    if (categoriesJSON) {
        slides.value.categories = JSON.parse(categoriesJSON);
    }
    if (collectionsJSON) {
        slides.value.collections = JSON.parse(collectionsJSON);
    }
})
</script>

<template>
    <div class="slides-forms">
        <div v-for="(slide, index) in slides[props.type]" :key="`slide_${slide.id}_${type}`" class="slides-forms__form">
            <div class="header">
                <span>{{ index + 1 }}</span>
                <button @click="removeSlide(slide.id)">X</button>
            </div>
            <textarea v-model="slides[props.type][index].text" placeholder="Текст на слайде" />
            <input v-model="slides[props.type][index].link" placeholder="Ссылка, куда ведет слайд" />
            <input v-model="slides[props.type][index].img" placeholder="Ссылка на изображение" />
        </div>
        <button @click="createEmptySlide">Добавить</button>
    </div>

    <div class="slides-forms">
        <button class="button-dark" @click="saveResultToLocalStorage">Сохранить</button>
        <button @click="reset">Очистить</button>
        <button @click="resetAll">Очистить всё</button>
    </div>

    <div class="result">
        <h3>Результат</h3>
        <pre>{{ preview }}</pre>
        <button @click="copyToClipboard">Копировать в буфер</button>
    </div>
</template>

<style scoped>
.slides-forms {
    display: flex;
    flex-wrap: wrap;
    gap: 24px;
    margin-top: 16px;
}

.slides-forms__form {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding: 24px;
    border-radius: 8px;
    border: 2px solid #646cff;
}

.slides-forms__form input, .slides-forms__form textarea {
    border: 2px solid transparent;
    max-width: 190px;
    background-color: #f5f5f5;
    color: #242424;
    padding: 8px 16px;
    min-height: 40px;
    border-radius: 8px;
    outline: none;
    line-height: 1.15;
    overflow: hidden;
    transition: all .25s;
    font-family: sans-serif;
}

.slides-forms__form textarea {
    height: 128px;
    resize: none;
}

.slides-forms__form input:focus {
    border-color: #646cff;
}

.slides-forms__form input:hover {
    border-color: rgba(100, 108, 255, 0.5);
}

.slides-forms__form .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    gap: 6px;
    font-weight: 600;
}

.result {
    margin-top: 24px;
    padding: 16px;
    position: relative;
}

.result pre {
    white-space: pre-wrap;
    word-wrap: break-word;
    padding: 16px;
    border-radius: 8px;
    background-color: #f5f5f5;
}

.button-dark {
    background-color: #646cff;
    color: white;
    cursor: pointer;
    transition: background-color 0.25s;
}

.button-dark:hover {
    background-color: #535bf2;
}
</style>
