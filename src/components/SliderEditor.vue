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

const isModalVisible = ref(false);
const codeFromSite = ref('');

const extractArrayFromString = (str: string) => {
    // Удаляем лишние пробелы, переносы строк и точку с запятой в конце
    const cleanedStr = str.replace(/[\n\t]/g, '').replace(/;\s*$/, '').trim();

    // Извлекаем часть с массивом (удаляем "const varName =")
    const arrayPart = cleanedStr.replace(/^const\s+\w+\s*=\s*/, '');

    try {
        // Выполняем код и возвращаем массив (безопаснее, чем eval)
        return new Function(`return ${arrayPart}`)();
    } catch (e) {
        console.error('Error parsing array:', e);
        return null;
    }
}

const saveCodeFromSite = () => {
    const obj = extractArrayFromString(codeFromSite.value);

    if (obj) {
        slides.value[props.type] = obj;
        isModalVisible.value = false;
        codeFromSite.value = '';
        saveResultToLocalStorage();
    } else {
        alert('Ошибка обработки данных');
    }
};
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
        <div class="result__title">
            <h3>Результат</h3>
            <button @click="isModalVisible = true">Вставить контент с сайта</button>
        </div>
        <pre>{{ preview }}</pre>
        <button @click="copyToClipboard">Копировать в буфер</button>
    </div>

    <teleport to="body">
        <Transition>
            <div v-if="isModalVisible" class="modal-wrapper">
                <button class="close" @click="isModalVisible = false">X</button>

                <div class="modal">
                    <h4>Вставка кода с сайта для редактирования</h4>
                    <textarea v-model="codeFromSite" placeholder="Вставь код между засечек с сайта" />
                    <button @click="saveCodeFromSite">Сохранить</button>
                </div>
            </div>
        </Transition>
    </teleport>
</template>

<style scoped>
.v-enter-active,
.v-leave-active {
    transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
    opacity: 0;
}

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

input, textarea {
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

textarea {
    height: 128px;
    resize: none;
}

input:focus {
    border-color: #646cff;
}

input:hover {
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
    display: flex;
    flex-direction: column;
    align-items: start;
    margin-top: 24px;
    padding: 16px;
    position: relative;
}

.result__title {
    display: flex;
    gap: 8px;
    align-items: center;
}

.result pre {
    width: 100%;
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

.modal-wrapper {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-wrapper .close {
    position: absolute;
    top: 16px;
    right: 16px;
}

.modal {
    display: flex;
    flex-direction: column;
    width: 50vw;
    min-width: 480px;
    background-color: white;
    border-radius: 16px;
    padding: 16px;

    h4 {
        margin: 0 0 16px;
    }

    textarea {
        height: 320px;
        max-height: unset;
        max-width: unset;
        margin-bottom: 16px;
    }
}

</style>
