<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
// @ts-ignore
import TypeIt from 'typeit'
import Markdown from 'markdown-it'
import type { SlidevMarkdown } from '@slidev/types'
// @ts-ignore
import Cover from '@slidev/theme-default/layouts/cover.vue'
import { parse } from '@slidev/parser'
import Default from '@slidev/client/layouts/default.vue'
import Center from '@slidev/client/layouts/center.vue'
import SlideContainer from '@slidev/client/internals/SlideContainer.vue'
import '@slidev/client/styles/layouts-base.css'
import '@slidev/theme-default/styles/layouts.css'

const page = ref(0)
const paused = ref(false)
const code = ref('')
const info = ref<SlidevMarkdown>()
const block = ref<HTMLPreElement>()

const markdown = new Markdown()

function getLayout(id: number) {
  const name = info.value?.slides?.[id]?.frontmatter?.layout || 'default'
  return name === 'cover'
    ? Cover
    : name === 'center'
      ? Center
      : Default
}

watch([code, paused], () => {
  if (paused.value)
    return
  try {
    info.value = parse(code.value)
  }
  catch (e) {

  }
})

function getAttrs(id: number) {
  return info.value?.slides?.[id]?.frontmatter
}

function getHTML(id: number) {
  const content = info?.value?.slides?.[id]?.content
  if (!content)
    return ''
  return markdown.render(content)
}

function pause() {
  paused.value = true
}
function resume() {
  paused.value = false
}

const COVER_URL = 'https://sli.dev/demo-cover.png'
if (typeof window !== 'undefined') {
  const img1 = new Image()
  img1.src = COVER_URL
}

onMounted(() => {
  new TypeIt(block.value, {
    speed: 50,
    startDelay: 900,
    afterStep: () => {
      code.value = JSON.parse(JSON.stringify(block.value!.innerText.replace('|', '')))
    },
  })
    .type('<br><span class="token title"># Bienvenue chez Slidev!</span><br><br>', { delay: 400 })
    .type('Diapositives de présentation pour Développeurs', { delay: 400 })
    .move('START', { speed: 0 })
    .type('<br>')
    .move('START')
    .exec(pause)
    .type('<span class="token punctuation">---<br><br>---</span>')
    .move(-4)
    .type('<span class="token tag">layout:</span> center')
    .exec(resume)
    .pause(1000)
    .exec(pause)
    .delete(6, { delay: 100, speed: 50 })
    .type('cover')
    .exec(resume)
    .exec(pause)
    .pause(1000)
    .type('<br>')
    .type('<span class="token tag">background:</span> ', { delay: 200 })
    .type(COVER_URL, { speed: 0 })
    .exec(resume)
    .pause(1000)
    .move('END', { speed: 0 })
    .exec(pause)
    .type('<br><br><span class="token punctuation">---</span><br><br>', { delay: 400 })
    .exec(resume)
    .exec(() => setTimeout(() => page.value = 1))
    .type('<span class="token title"># Page 2</span><br><br>', { delay: 400 })
    .type('- 📄 Rédigez vos diapositives dans un unique fichier Markdown<br>', { delay: 800 })
    .type('- 🌈 Thèmes, blocs de code, composants interactifs<br>', { delay: 800 })
    .type('- 😎 Lisez la documentation pour en savoir plus !', { delay: 800 })
    .exec(() => setTimeout(() => page.value = 0))
    .go()
})
</script>

<template>
  <div>
    <DemoEditor>
      <div class="text-sm opacity-50">
        ./slides.md
      </div>

      <div class="language-md !bg-transparent">
        <pre ref="block" class="text-left whitespace-normal font-mono bg-transparent"></pre>
      </div>
    </DemoEditor>

    <DemoSlide class="text-left">
      <div
        class="flex h-full dark:bg-[#181819] transition-transform transform duration-500"
        style="width: 200%"
        :class="page === 1 ? '-translate-x-1/2': ''"
      >
        <SlideContainer class="w-full h-full">
          <component :is="getLayout(0)" v-bind="getAttrs(0)">
            <div v-html="getHTML(0)"></div>
          </component>
        </SlideContainer>
        <SlideContainer class="w-full h-full">
          <component :is="getLayout(1)" v-bind="getAttrs(1)">
            <div v-html="getHTML(1)"></div>
          </component>
        </SlideContainer>
      </div>
      <div
        class="absolute left-2 bottom-1 flex text-gray-400"
        opacity="0 hover:100"
      >
        <div class="icon-btn" :class="{ disabled: page === 0 }" @click="page = 0">
          <carbon:chevron-left />
        </div>
        <div class="icon-btn" :class="{ disabled: page === 1 }" @click="page = 1">
          <carbon:chevron-right />
        </div>
      </div>
    </DemoSlide>
  </div>
</template>

<style>
.slidev-layout ul {
  padding: 0;
}
.slidev-layout li {
  line-height: 2.4em;
}
</style>
