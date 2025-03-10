<script setup lang="ts">
  import type { NavItem } from '@nuxt/content'

const route = useRoute()
const root = route.path.split('/').filter(Boolean)[0];
const nav = inject<Ref<NavItem[]>>('navigation')
let navigation = nav;
if (root == 'framework') {
  const pkg = route.path.split('/').filter(Boolean)[1];
//  navigation = computed(() => nav!.value[1].children.find(item => item._path.startsWith('/framework/'+pkg))?.children || [])
  navigation = computed(() => nav!.value[1].children || [])
} else if (root == 'tutorials') {
  navigation = computed(() => nav.value.filter(item => item._path.startsWith('/tutorials')))
} else if (root == 'geniebuilder') {
  navigation = computed(() => nav!.value[0].children.find(item => item._path.startsWith('/geniebuilder/'))?.children || [])
}



import { withoutTrailingSlash } from 'ufo'

definePageMeta({
  layout: 'framework'
})

const { toc, seo } = useAppConfig()

const { data: page } = await useAsyncData(route.path, () => queryContent(route.path).findOne())
if (!page.value) {
  throw createError({ statusCode: 404, statusMessage: 'Page not found', fatal: true })
}

const { data: surround } = await useAsyncData(`${route.path}-surround`, () => queryContent()
  .where({ _extension: 'md', navigation: { $ne: false } })
  .only(['title', 'description', '_path'])
  .findSurround(withoutTrailingSlash(route.path))
)

useSeoMeta({
  title: page.value.title,
  ogTitle: `${page.value.title} - ${seo?.siteName}`,
  description: page.value.description,
  ogDescription: page.value.description
})

/*
defineOgImage({
  component: 'Docs',
  title: page.value.title,
  description: page.value.description
})
*/

const headline = computed(() => findPageHeadline(page.value))

const links = computed(() => [toc?.bottom?.edit && {
  icon: 'i-heroicons-pencil-square',
  label: 'Edit this page',
  to: `${toc.bottom.edit}/${page?.value?._file}`,
  target: '_blank'
}, ...(toc?.bottom?.links || [])].filter(Boolean))


// popover
const isOpen = ref(false)
</script>

<template>
  <UContainer>
  <UPage>

  <template #left v-if="root!='tutorials'">
    <UAside >
    <!--<PkgSelect v-if="root== 'framework'"/> -->
    <UNavigationTree :links="mapContentNavigation(navigation)" :multiple="true" default-open style="min-height:600px" :defaultOpen="1"/>
    </UAside>
  </template>

  <UPageBody prose>
  <ContentRenderer
      v-if="page.body"
      :value="page"
      />

  <hr v-if="surround?.length">

  <UContentSurround :surround="surround" />
  </UPageBody>

  <template
      v-if="page.toc !== false"
      #right
      >
      <UContentToc
          :title="toc?.title"
          :links="page.body?.toc?.links"
          >
          <template
              v-if="toc?.bottom"
              #bottom
              >
              <div
                  class="hidden lg:block space-y-6"
                  :class="{ '!mt-6': page.body?.toc?.links?.length }"
                  >
                  <UDivider
                      v-if="page.body?.toc?.links?.length"
                      type="dashed"
                      />

                  <UPageLinks
                      :title="toc.bottom.title"
                      :links="links"
                      />

            <div style="margin-top:4px">
            <UButton label="Talk to us" @click="isOpen = true" icon="i-heroicons-user-group" variant="link" :padded="false"/>
            </div>

            <UModal v-model="isOpen" color="secondary">
            <div class="p-4">
              <iframe style="width:100%;height:620px" src="https://meetings.hubspot.com/pere-gimenez?embed=true"></iframe>
            </div>
            </UModal>

              </div>

          </template>
      </UContentToc>
  </template>
  </UPage>
  </UContainer>
</template>
