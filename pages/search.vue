<template>
  <div>
    <LoaderComponent v-if="loading" />
    <div v-else>
      <HeaderComponent title="Ricerca" :breadcrumb="breadcrumb" />
      <div class="flex flex-wrap items-stretch gap-4">
        <div
          v-for="post in posts"
          :key="post.id"
          class="flex flex-col transition-all duration-300 bg-white border rounded-md w-80 hover:shadow-md hover:scale-105 border-zinc-200"
        >
          <div class="h-full p-2 text-lg font-semibold text-secondary">{{ post.title }}</div>
          <div
            class="bottom-0 px-2 py-3 font-semibold text-center text-white transition-all duration-300 rounded-b-lg cursor-pointer h-fit bg-secondary hover:cursor-pointer hover:bg-blue-800"
            @click="goToPost(post)"
          >
            Apri dettaglio
          </div>
        </div>
      </div>

      <div v-if="employees.length" class="flex flex-col gap-2 pt-4">
        <div class="text-xl font-semibold text-secondary">Dipendenti</div>
        <div v-for="employee in employees" :key="employee.id">
          <EmployeeCard :employee="employee" />
        </div>
      </div>

      <div v-if="media.length" class="flex flex-col gap-2 pt-4">
        <div class="text-xl font-semibold text-secondary">Media</div>
        <div v-for="m in media" :key="m.id">
          <AttachmentComponent
            :title="m.title"
            :url="m.url"
            :type="m.type"
            :size="m.size"
            :categories="m.categories"
            :tags="m.tags"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
  import LoaderComponent from '~/components/common/LoaderComponent.vue'
  import HeaderComponent from '~/components/common/HeaderComponent.vue'
  import AttachmentComponent from '~/components/common/AttachmentComponent.vue'
  import EmployeeCard from '~/components/common/EmployeeCard.vue'

  import { CUSTOM_POST_TYPES } from '~/utils/staticData/constants'
  import { formatArrayOfEmployees } from '~/utils'

  import { useCategoriesStore } from '~/stores/categories'
  import { useTagsStore } from '~/stores/tags'

  import { getPostById } from '~/api/posts'
  import { getEmployeeById } from '~/api/employees'
  import { mediaGlobalSearchAPI, postsGlobalSearchAPI } from '~/api/globalSearch'

  const route = useRoute()
  const categoriesStore = useCategoriesStore()
  const tagsStore = useTagsStore()

  const title = 'Ricerca'
  const loading = ref(false)
  const categories = ref([])
  const tags = ref([])
  const posts = ref([])
  const employees = ref([])
  const media = ref([])

  const breadcrumb = ref([
    {
      title: 'Home',
      slug: '/',
    },
    {
      title: title,
      slug: '/search',
    },
  ])

  watch(
    () => route.query,
    async () => {
      await runSearch()
    }
  )

  const goToPost = async (post) => {
    try {
      if (CUSTOM_POST_TYPES.includes(post.subtype)) await navigateTo(`${post.url}`, { id: post.id })
      else {
        const resPost = await getPostById(post.id, categories.value, tags.value)
        if (resPost?.categories[0]?.link)
          await navigateTo({
            name: `${resPost.categories[0].link}-slug`,
            params: { slug: resPost.slug },
            query: { id: post.id },
          })
      }
    } catch (error) {
      console.error(error)
    }
  }

  const runSearch = async () => {
    try {
      loading.value = true
      if (route.query) {
        employees.value = []
        posts.value = []
        media.value = []
        categories.value = []
        tags.value = []

        categories.value = await categoriesStore.getCategories()
        tags.value = await tagsStore.getTags()
        // results.value = await runGlobalSearchAPI(route.query.query, categories.value, tags.value)
        media.value = await mediaGlobalSearchAPI(route.query.query, categories.value, tags.value)
        const allPosts = await postsGlobalSearchAPI(route.query.query)
        posts.value = allPosts.filter((post) => !post.url.startsWith('/employee'))
        const allEmployees = allPosts.filter((post) => post.url.startsWith('/employee'))
        await Promise.all(
          allEmployees.map(async (e) => {
            const res = await getEmployeeById(e.id)
            employees.value.push(res.value)
          })
        )
        employees.value = formatArrayOfEmployees(employees.value)
      }
    } catch (err) {
      console.error(err)
    } finally {
      loading.value = false
    }
  }

  onMounted(async () => {
    await runSearch()
  })
</script>
