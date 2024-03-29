<script setup lang="ts">
import { useBlogStore } from '@/stores/blog';
import type { Post } from '@/types/post';
import { computed, onMounted, ref, type Ref } from 'vue';
import { useRoute, useRouter } from 'vue-router'

const route = useRoute();
const router = useRouter();
const id = Number(route.params.id);
const store = useBlogStore();
const blog: Ref<Post | undefined> = ref(store.getBlog(id));

const date = computed(() => {
    if (!blog.value) return undefined
    return blog.value?.createAt.split(' ')[0]
})
const daysBetween = computed(() => {
    if (!blog.value) return undefined
    let dateString = blog.value?.createAt.split(' ')[0];
    const date = new Date(dateString);
    const today = new Date();
    const msBetween = today.getTime() - date.getTime();
    const daysBetween = Math.round(msBetween / (1000 * 60 * 60 * 24));
    return daysBetween;
})
const time = computed(() => {
    return blog.value?.createAt.split(' ')[1]
})

const toPersianString = (str: string): string => {

    const persianDigits = ['۰', '۱', '۲', '۳', '۴', '۵', '۶', '۷', '۸', '۹'];
    let output = '';

    for (let i = 0; i < str.length; i++) {
        let char = str[i];
        if (!isNaN(parseInt(char))) {
            output += persianDigits[parseInt(char)];
        } else {
            output += char;
        }
    }
    return output;
}

onMounted(() => {
    if (!blog.value) {
        router.push("/404")
    }
})

</script>

<template>
    <div v-if="blog" dir="rtl" class="relative font-sahel">
        <div class="z-50 container max-w-[760px] bg-bg-color-2 pb-6 sm:px-6 px-3 pt-10 w-full mx-auto">
            <div class="z-50 relative rounded-[16px] overflow-hidden">
                <img class="w-full blog-main-image h-auto aspect-[16/9] flex justify-center text-white object-cover"
                    :src="blog.imageURL" :alt="blog.imageAlt">
            </div>

            <div class="mt-4 p-2">
                <div class="mb-3 flex gap-3">
                    <RouterLink to="" v-for="categoryItem in blog?.categories" :key="categoryItem.id"
                        :style="{ background: `-webkit-linear-gradient(315deg, ${categoryItem.color[0]} 25%, ${categoryItem.color[1]})` }"
                        class="font-semibold text-sm category">
                        {{ categoryItem.title }}
                    </RouterLink>
                </div>
                <h1 class="mb-4 text-3xl font-extrabold">
                    {{ blog?.title }}
                </h1>
                <div class="blog-content text-base font-sahel font-normal" v-html="blog?.body">
                </div>
            </div>
            <div class="absolute top-0 right-0 left-0 w-full max-h-[240px] h-auto aspect-[16/5]"
                :style="{ background: `${blog?.blogColor}` }">
            </div>
        </div>
        <div class="mx-auto container max-w-[760px] py-3 px-2 flex justify-end text-sm font-extrabold text-text-color-2">
            <time :datetime="date">
                منتشر شده در
                {{ toPersianString(`${daysBetween}`) }}
                روز پیش
            </time>
        </div>

    </div>
</template>

<style scoped>
.category {
    background-clip: border-box;
    background-clip: text !important;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}
</style>