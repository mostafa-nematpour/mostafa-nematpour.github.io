<script setup lang="ts">
import { useBlogStore } from '@/stores/blog'
import type { Post } from '@/types/post'
import { computed, onMounted, ref, type Ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import MainHeader from '~/components/header/MainHeader.vue'
import CommentForm from '~/components/blog/CommentForm.vue'
import CommentList from '~/components/blog/CommentList.vue'
import SVGBack from '~/components/ui/svg/SVGBack.vue'

const route = useRoute()
const router = useRouter()
const id = Number(route.params.id)
const store = useBlogStore()
const blog: Ref<Post | undefined> = ref(store.getBlog(id))

const date = computed(() => {
  if (!blog.value) return undefined
  return blog.value?.createAt.split(' ')[0]
})
const daysBetween = computed(() => {
  if (!blog.value) return undefined
  let dateString = blog.value?.createAt.split(' ')[0]
  const date = new Date(dateString)
  const today = new Date()
  const msBetween = today.getTime() - date.getTime()
  const daysBetween = Math.round(msBetween / (1000 * 60 * 60 * 24))
  return daysBetween
})
const time = computed(() => {
  return blog.value?.createAt.split(' ')[1]
})

const toPersianString = (str: string): string => {
  const persianDigits = ['۰', '۱', '۲', '۳', '۴', '۵', '۶', '۷', '۸', '۹']
  let output = ''

  for (let i = 0; i < str.length; i++) {
    let char = str[i]
    if (!isNaN(parseInt(char))) {
      output += persianDigits[parseInt(char)]
    } else {
      output += char
    }
  }
  return output
}

const comments = ref<InstanceType<typeof CommentList> | null>(null)
const refreshCommentList = () => {
  comments.value?.refreshCommentList()
}

onMounted(() => {
  if (!blog.value) {
    router.push('/404')
  }
})

useSeoMeta({
  title: blog.value?.title,
  ogTitle: blog.value?.title,
  description: blog.value?.description,
  ogDescription: blog.value?.description,
  ogImage: blog.value?.imageURL,
  twitterCard: 'summary_large_image'
})
</script>

<template>
  <MainHeader>
    <template v-slot:options>
      <RouterLink title="پست‌ها" to="/posts"
        class="ms-3 p-[-20px] m-1 transition ease-in-out delay-150 hover:bg-c-bg-soft flex items-center justify-center rounded-full aspect-square w-auto">
        <SVGBack></SVGBack>
      </RouterLink>
    </template>
  </MainHeader>
  <div v-if="blog" dir="rtl" class="relative font-sahel post-page">
    <div class="bg-c-bg z-50 container max-w-[47.5rem] pb-6 sm:px-6 px-3 pt-1 w-full mx-auto">
      <h1 class="mb-4 text-c-text-3 text-4xl leading-normal font-extrabold relative z-[51]">
        {{ blog?.title }}
      </h1>
      <div v-if="blog?.imageURL" class="z-50 relative rounded-[16px] overflow-hidden">
        <img class="w-full blog-main-image h-auto aspect-[16/9] flex justify-center text-white object-cover"
          :src="blog.imageURL" :alt="blog?.imageAlt ?? ''" />
      </div>
      <div class="mt-4 p-2">
        <div class="mb-3 flex gap-3">
          <RouterLink to="" v-for="categoryItem in blog?.categories" :key="categoryItem.id" :style="{
    background: `-webkit-linear-gradient(315deg, ${categoryItem.color[0]} 25%, ${categoryItem.color[1]})`
  }" class="font-semibold text-sm category">
            {{ categoryItem.title }}
          </RouterLink>
        </div>

        <div class="blog-content text-base font-sahel font-normal text-c-text" v-html="blog?.body"></div>
      </div>
    </div>
    <div
      class="mx-auto container max-w-[47.5rem] py-3 sm:px-6 px-3  flex justify-end text-sm font-extrabold text-text-color-2">
      <time :datetime="date">
        منتشر شده در
        {{ toPersianString(`${daysBetween}`) }}
        روز پیش
      </time>
    </div>
    <div>
      <CommentForm :postId="blog?.id" @added="refreshCommentList"
        class="mx-auto container max-w-[47.5rem] py-3 sm:px-6 px-3 flex justify-end text-sm font-extrabold" />
      <CommentList :postId="blog?.id" ref="comments"
        class="mx-auto container max-w-[47.5rem] py-3 sm:px-6 px-3 justify-end text-sm font-extrabold text-text-color-2" />
    </div>
  </div>
</template>

<style>
.post-page .category {
  background-clip: border-box;
  background-clip: text !important;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>
