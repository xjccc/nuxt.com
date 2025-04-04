<script setup lang="ts">
import { kebabCase } from 'scule'
import type { ContentNavigationItem } from '@nuxt/content'
import { findPageBreadcrumb, mapContentNavigation } from '#ui-pro/utils'

definePageMeta({
  layout: 'docs',
  heroBackground: 'opacity-30'
})

const navigation = inject<Ref<ContentNavigationItem[]>>('navigation', ref([]))

const route = useRoute()

const { data: page } = await useAsyncData(kebabCase(route.path), () => queryCollection('docs').path(route.path).first())
if (!page.value) {
  throw createError({ statusCode: 404, statusMessage: 'Page not found', fatal: true })
}

const { data: surround } = await useAsyncData(`${kebabCase(route.path)}-surround`, () => {
  return queryCollectionItemSurroundings('docs', route.path, {
    fields: ['description']
  })
})

const breadcrumb = computed(() => {
  const links = mapContentNavigation(findPageBreadcrumb(navigation.value, page.value)).map(link => ({
    label: link.label,
    to: link.to
  }))

  if (route.path.startsWith('/docs/bridge') || route.path.startsWith('/docs/migration')) {
    links.splice(1, 0, {
      label: 'Upgrade Guide',
      to: '/docs/getting-started/upgrade'
    })
  }

  return links
})

const titleTemplate = computed(() => findTitleTemplate(page, navigation))

const editLink = computed(() => `https://github.com/nuxt/nuxt/edit/main/docs/${page?.value?.stem?.split('/').slice(1).join('/')}.${page?.value?.extension}`)

const communityLinks = computed(() => [{
  icon: 'i-lucide-heart',
  label: 'Become a Sponsor',
  to: 'https://go.nuxt.com/sponsor',
  target: '_blank'
}, {
  icon: 'i-lucide-chef-hat',
  label: 'Master Nuxt',
  to: 'https://masteringnuxt.com/nuxt3',
  target: '_blank'
}, {
  icon: 'i-lucide-award',
  label: 'Nuxt Certification',
  to: 'https://certification.nuxt.com',
  target: '_blank'
}])

const title = page.value.seo?.title || page.value.title
const description = page.value.seo?.description || page.value.description

useSeoMeta({
  titleTemplate,
  title,
  description,
  ogDescription: description,
  ogTitle: titleTemplate.value?.includes('%s') ? titleTemplate.value.replace('%s', title) : title
})

defineOgImageComponent('Docs', {
  headline: breadcrumb.value.length ? breadcrumb.value.map(link => link.label).join(' > ') : '',
  title,
  description
})
</script>

<template>
  <UPage v-if="page">
    <UPageHeader v-bind="page" :links="page.links?.map(link => ({ ...link, size: 'md' }))">
      <template #headline>
        <UBreadcrumb :items="breadcrumb" />
      </template>
    </UPageHeader>

    <UPageBody>
      <ContentRenderer v-if="page.body" :value="page" />
      <div>
        <USeparator class="my-10">
          <div class="flex items-center gap-2 text-sm dark:text-gray-400">
            <UButton size="sm" variant="link" color="neutral" to="https://github.com/nuxt/nuxt/issues/new/choose" target="_blank">
              Report an issue
            </UButton>
            or
            <UButton size="sm" variant="link" color="neutral" :to="editLink" target="_blank">
              Edit this page on GitHub
            </UButton>
          </div>
        </USeparator>
        <UContentSurround :surround="surround" />
      </div>
    </UPageBody>

    <template v-if="page?.body?.toc?.links?.length" #right>
      <UContentToc :links="page.body?.toc?.links" highlight class="lg:backdrop-blur-none">
        <template #bottom>
          <div class="hidden lg:block space-y-6" :class="{ '!mt-6': page.body?.toc?.links?.length }">
            <USeparator v-if="page.body?.toc?.links?.length" type="dashed" />
            <UPageLinks title="Community" :links="communityLinks" />
            <USeparator type="dashed" />
            <SocialLinks />
            <Ads />
          </div>
        </template>
      </UContentToc>
    </template>
  </UPage>
</template>
