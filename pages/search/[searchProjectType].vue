<template>
  <div
    :class="{
      'search-page': true,
      'normal-page': true,
      'alt-layout': $cosmetics.searchLayout,
    }"
  >
    <aside
      :class="{
        'normal-page__sidebar': true,
        open: sidebarMenuOpen,
      }"
      aria-label="Filters"
    >
      <section class="card filters-card" role="presentation">
        <div class="sidebar-menu" :class="{ 'sidebar-menu_open': sidebarMenuOpen }">
          <button
            :disabled="
              onlyOpenSource === false &&
              selectedEnvironments.length === 0 &&
              selectedVersions.length === 0 &&
              facets.length === 0 &&
              orFacets.length === 0
            "
            class="iconified-button"
            @click="clearFilters"
          >
            <ClearIcon aria-hidden="true" />
            Clear filters
          </button>
          <section aria-label="Category filters">
            <div v-for="(categories, header) in categoriesMap" :key="header">
              <h3
                v-if="categories.filter((x) => x.project_type === projectType.actual).length > 0"
                class="sidebar-menu-heading"
              >
                {{ $formatCategoryHeader(header) }}
              </h3>

              <SearchFilter
                v-for="category in categories.filter((x) => x.project_type === projectType.actual)"
                :key="category.name"
                :active-filters="facets"
                :display-name="$formatCategory(category.name)"
                :facet-name="`categories:'${encodeURIComponent(category.name)}'`"
                :icon="header === 'resolutions' ? null : category.icon"
                @toggle="toggleFacet"
              />
            </div>
          </section>
          <section
            v-if="projectType.id !== 'resourcepack' && projectType.id !== 'datapack'"
            aria-label="Loader filters"
          >
            <h3
              v-if="
                $tag.loaders.filter((x) => x.supported_project_types.includes(projectType.actual))
                  .length > 0
              "
              class="sidebar-menu-heading"
            >
              Loaders
            </h3>
            <SearchFilter
              v-for="loader in $tag.loaders.filter((x) => {
                if (
                  projectType.id === 'mod' &&
                  !showAllLoaders &&
                  x.name !== 'forge' &&
                  x.name !== 'fabric' &&
                  x.name !== 'quilt'
                ) {
                  return false
                } else if (projectType.id === 'mod' && showAllLoaders) {
                  return $tag.loaderData.modLoaders.includes(x.name)
                } else if (projectType.id === 'plugin') {
                  return $tag.loaderData.pluginLoaders.includes(x.name)
                } else if (projectType.id === 'datapack') {
                  return $tag.loaderData.dataPackLoaders.includes(x.name)
                } else {
                  return x.supported_project_types.includes(projectType.actual)
                }
              })"
              :key="loader.name"
              ref="loaderFilters"
              :active-filters="orFacets"
              :display-name="$formatCategory(loader.name)"
              :facet-name="`categories:'${encodeURIComponent(loader.name)}'`"
              :icon="loader.icon"
              @toggle="toggleOrFacet"
            />
            <Checkbox
              v-if="projectType.id === 'mod'"
              v-model="showAllLoaders"
              :label="showAllLoaders ? 'Less' : 'More'"
              description="Show all loaders"
              style="margin-bottom: 0.5rem"
              :border="false"
              :collapsing-toggle-style="true"
            />
          </section>
          <section v-if="projectType.id === 'plugin'" aria-label="Platform loader filters">
            <h3
              v-if="
                $tag.loaders.filter((x) => x.supported_project_types.includes(projectType.actual))
                  .length > 0
              "
              class="sidebar-menu-heading"
            >
              Proxies
            </h3>
            <SearchFilter
              v-for="loader in $tag.loaders.filter((x) =>
                $tag.loaderData.pluginPlatformLoaders.includes(x.name)
              )"
              :key="loader.name"
              ref="platformFilters"
              :active-filters="orFacets"
              :display-name="$formatCategory(loader.name)"
              :facet-name="`categories:'${encodeURIComponent(loader.name)}'`"
              :icon="loader.icon"
              @toggle="toggleOrFacet"
            />
          </section>
          <section
            v-if="!['resourcepack', 'plugin', 'shader', 'datapack'].includes(projectType.id)"
            aria-label="Environment filters"
          >
            <h3 class="sidebar-menu-heading">Environments</h3>
            <SearchFilter
              :active-filters="selectedEnvironments"
              display-name="Client"
              facet-name="client"
              @toggle="toggleEnv"
            >
              <ClientIcon aria-hidden="true" />
            </SearchFilter>
            <SearchFilter
              :active-filters="selectedEnvironments"
              display-name="Server"
              facet-name="server"
              @toggle="toggleEnv"
            >
              <ServerIcon aria-hidden="true" />
            </SearchFilter>
          </section>
          <h3 class="sidebar-menu-heading">Minecraft versions</h3>
          <Checkbox
            v-model="showSnapshots"
            label="Include snapshots"
            description="Include snapshots"
            style="margin-bottom: 0.5rem"
            :border="false"
          />
          <multiselect
            v-model="selectedVersions"
            :options="
              showSnapshots
                ? $tag.gameVersions.map((x) => x.version)
                : $tag.gameVersions
                    .filter((it) => it.version_type === 'release')
                    .map((x) => x.version)
            "
            :multiple="true"
            :searchable="true"
            :show-no-results="false"
            :close-on-select="false"
            :clear-search-on-select="false"
            :show-labels="false"
            :selectable="() => selectedVersions.length <= 6"
            placeholder="Choose versions..."
            @update:model-value="onSearchChange(1)"
          />
          <h3 class="sidebar-menu-heading">Open source</h3>
          <Checkbox
            v-model="onlyOpenSource"
            label="Open source only"
            style="margin-bottom: 0.5rem"
            :border="false"
            @update:model-value="onSearchChange(1)"
          />
        </div>
      </section>
    </aside>
    <section class="normal-page__content">
      <div
        v-if="projectType.id === 'modpack' && $orElse($cosmetics.modpacksAlphaNotice, true)"
        class="card information"
        aria-label="Information"
      >
        Modpack support is currently in alpha, and modpacks can only be created and installed
        through third party tools. Our documentation includes instructions on
        <a href="https://docs.modrinth.com/docs/modpacks/playing_modpacks/" :target="$external()"
          >playing modpacks</a
        >
        with
        <a rel="noopener" href="https://atlauncher.com/about" :target="$external()">ATLauncher</a>,
        <a rel="noopener" href="https://multimc.org/" :target="$external()">MultiMC</a>, and
        <a rel="noopener" href="https://prismlauncher.org" :target="$external()"> Prism Launcher</a
        >. Pack creators can reference our documentation on
        <a href="https://docs.modrinth.com/docs/modpacks/creating_modpacks/" :target="$external()"
          >creating modpacks</a
        >. Join us on
        <a rel="noopener" href="https://discord.gg/EUHuJHt" :target="$external()">Discord</a>
        for support.
      </div>
      <Promotion />
      <div v-if="!$cosmetics.hideAds" class="server-banner">
        <h2>The hottest online Minecraft servers are waiting for you!</h2>
        <nuxt-link class="iconified-button brand-button" to="/frog">Check it out</nuxt-link>
      </div>
      <div class="card search-controls">
        <div class="search-filter-container">
          <button
            class="iconified-button sidebar-menu-close-button"
            :class="{ open: sidebarMenuOpen }"
            @click="sidebarMenuOpen = !sidebarMenuOpen"
          >
            <FilterIcon aria-hidden="true" />
            Filters...
          </button>
          <div class="iconified-input">
            <label class="hidden" for="search">Search</label>
            <SearchIcon aria-hidden="true" />
            <input
              id="search"
              v-model="query"
              type="search"
              name="search"
              :placeholder="`Search ${projectType.display}s...`"
              autocomplete="off"
              @input="onSearchChange(1)"
            />
          </div>
        </div>
        <div class="sort-controls">
          <div class="labeled-control">
            <span class="labeled-control__label">Sort by</span>
            <Multiselect
              v-model="sortType"
              placeholder="Select one"
              class="search-controls__sorting labeled-control__control"
              track-by="display"
              label="display"
              :options="sortTypes"
              :searchable="false"
              :close-on-select="true"
              :show-labels="false"
              :allow-empty="false"
              @update:model-value="onSearchChange(1)"
            >
              <template #singleLabel="{ option }">
                {{ option.display }}
              </template>
            </Multiselect>
          </div>
          <div class="labeled-control">
            <span class="labeled-control__label">Show per page</span>
            <Multiselect
              v-model="maxResults"
              placeholder="Select one"
              class="labeled-control__control"
              :options="maxResultsForView[$cosmetics.searchDisplayMode[projectType.id]]"
              :searchable="false"
              :close-on-select="true"
              :show-labels="false"
              :allow-empty="false"
              @update:model-value="onMaxResultsChange(currentPage)"
            />
          </div>
          <button
            v-tooltip="$capitalizeString($cosmetics.searchDisplayMode[projectType.id]) + ' view'"
            :aria-label="$capitalizeString($cosmetics.searchDisplayMode[projectType.id]) + ' view'"
            class="square-button"
            @click="cycleSearchDisplayMode()"
          >
            <GridIcon v-if="$cosmetics.searchDisplayMode[projectType.id] === 'grid'" />
            <ImageIcon v-else-if="$cosmetics.searchDisplayMode[projectType.id] === 'gallery'" />
            <ListIcon v-else />
          </button>
        </div>
      </div>
      <Pagination
        :page="currentPage"
        :count="pageCount"
        :link-function="(x) => getSearchUrl(x <= 1 ? 0 : (x - 1) * maxResults)"
        class="pagination-before"
        @switch-page="onSearchChange"
      />
      <LogoAnimated v-if="searchLoading && !noLoad"></LogoAnimated>
      <div v-else-if="results && results.hits && results.hits.length === 0" class="no-results">
        <p>No results found for your query!</p>
      </div>
      <div v-else class="search-results-container">
        <div
          id="search-results"
          class="project-list"
          :class="'display-mode--' + $cosmetics.searchDisplayMode[projectType.id]"
          role="list"
          aria-label="Search results"
        >
          <ProjectCard
            v-for="result in results?.hits"
            :id="result.slug ? result.slug : result.project_id"
            :key="result.project_id"
            :display="$cosmetics.searchDisplayMode[projectType.id]"
            :featured-image="result.featured_gallery ? result.featured_gallery : result.gallery[0]"
            :type="result.project_type"
            :author="result.author"
            :name="result.title"
            :description="result.description"
            :created-at="result.date_created"
            :updated-at="result.date_modified"
            :downloads="result.downloads.toString()"
            :follows="result.follows.toString()"
            :icon-url="result.icon_url"
            :client-side="result.client_side"
            :server-side="result.server_side"
            :categories="result.display_categories"
            :search="true"
            :show-updated-date="sortType.name !== 'newest'"
            :hide-loaders="['resourcepack', 'datapack'].includes(projectType.id)"
            :color="result.color"
          />
        </div>
      </div>
      <pagination
        :page="currentPage"
        :count="pageCount"
        :link-function="(x) => getSearchUrl(x <= 1 ? 0 : (x - 1) * maxResults)"
        class="pagination-after"
        @switch-page="onSearchChangeToTop"
      />
    </section>
  </div>
</template>
<script>
import Multiselect from 'vue-multiselect'
import ProjectCard from '~/components/ui/ProjectCard'
import Pagination from '~/components/ui/Pagination'
import SearchFilter from '~/components/ui/search/SearchFilter'
import Checkbox from '~/components/ui/Checkbox'
import LogoAnimated from '~/components/brand/LogoAnimated'

import ClientIcon from '~/assets/images/categories/client.svg'
import ServerIcon from '~/assets/images/categories/server.svg'

import SearchIcon from '~/assets/images/utils/search.svg'
import ClearIcon from '~/assets/images/utils/clear.svg'
import FilterIcon from '~/assets/images/utils/filter.svg'
import GridIcon from '~/assets/images/utils/grid.svg'
import ListIcon from '~/assets/images/utils/list.svg'
import ImageIcon from '~/assets/images/utils/image.svg'

import Promotion from '~/components/ads/Promotion.vue'

export default defineNuxtComponent({
  components: {
    LogoAnimated,
    Promotion,
    ProjectCard,
    Pagination,
    Multiselect,
    SearchFilter,
    Checkbox,
    ClientIcon,
    ServerIcon,
    SearchIcon,
    ClearIcon,
    FilterIcon,
    GridIcon,
    ListIcon,
    ImageIcon,
  },
  data() {
    return {
      previousMaxResults: 20,

      maxResultsForView: {
        list: [5, 10, 15, 20, 50, 100],
        grid: [6, 12, 18, 24, 48, 96],
        gallery: [6, 10, 16, 20, 50, 100],
      },

      sidebarMenuOpen: false,
      showAllLoaders: false,
    }
  },
  setup() {
    const data = useNuxtApp()
    const route = useRoute()

    const query = ref('')
    const facets = ref([])
    const orFacets = ref([])
    const selectedVersions = ref([])
    const onlyOpenSource = ref(false)
    const showSnapshots = ref(false)
    const selectedEnvironments = ref([])
    const sortTypes = shallowReadonly([
      { display: 'Relevance', name: 'relevance' },
      { display: 'Download count', name: 'downloads' },
      { display: 'Follow count', name: 'follows' },
      { display: 'Recently published', name: 'newest' },
      { display: 'Recently updated', name: 'updated' },
    ])
    const sortType = ref({ display: 'Relevance', name: 'relevance' })
    const maxResults = ref(20)
    const currentPage = ref(1)
    const projectType = ref({ id: 'mod', display: 'mod', actual: 'mod' })

    if (route.query.q) {
      query.value = route.query.q
    }
    if (route.query.f) {
      facets.value = getArrayOrString(route.query.f)
    }
    if (route.query.g) {
      orFacets.value = getArrayOrString(route.query.g)
    }
    if (route.query.v) {
      selectedVersions.value = getArrayOrString(route.query.v)
    }
    if (route.query.l) {
      onlyOpenSource.value = route.query.l === 'true'
    }
    if (route.query.h) {
      showSnapshots.value = route.query.h === 'true'
    }
    if (route.query.e) {
      selectedEnvironments.value = getArrayOrString(route.query.e)
    }
    if (route.query.s) {
      sortType.value.name = route.query.s

      switch (sortType.value.name) {
        case 'relevance':
          sortType.value.display = 'Relevance'
          break
        case 'downloads':
          sortType.value.display = 'Downloads'
          break
        case 'newest':
          sortType.value.display = 'Recently published'
          break
        case 'updated':
          sortType.value.display = 'Recently updated'
          break
        case 'follows':
          sortType.value.display = 'Follow count'
          break
      }
    }

    if (route.query.m) {
      maxResults.value = route.query.m
    }
    if (route.query.o) {
      currentPage.value = Math.ceil(route.query.o / maxResults.value) + 1
    }

    projectType.value = data.$tag.projectTypes.find(
      (x) => x.id === route.path.substring(1, route.path.length - 1)
    )

    const noLoad = ref(false)
    const {
      data: rawResults,
      refresh: refreshSearch,
      pending: searchLoading,
    } = useLazyFetch(
      () => {
        const config = useRuntimeConfig()
        const base = process.server ? config.apiBaseUrl : config.public.apiBaseUrl

        const params = [`limit=${maxResults.value}`, `index=${sortType.value.name}`]

        if (query.value.length > 0) {
          params.push(`query=${query.value.replace(/ /g, '+')}`)
        }

        if (
          facets.value.length > 0 ||
          orFacets.value.length > 0 ||
          selectedVersions.value.length > 0 ||
          selectedEnvironments.value.length > 0 ||
          projectType.value
        ) {
          let formattedFacets = []
          for (const facet of facets.value) {
            formattedFacets.push([facet])
          }

          // loaders specifier
          if (orFacets.value.length > 0) {
            formattedFacets.push(orFacets.value)
          } else if (projectType.value.id === 'plugin') {
            formattedFacets.push(
              data.$tag.loaderData.allPluginLoaders.map(
                (x) => `categories:'${encodeURIComponent(x)}'`
              )
            )
          } else if (projectType.value.id === 'mod') {
            formattedFacets.push(
              data.$tag.loaderData.modLoaders.map((x) => `categories:'${encodeURIComponent(x)}'`)
            )
          } else if (projectType.value.id === 'datapack') {
            formattedFacets.push(
              data.$tag.loaderData.dataPackLoaders.map(
                (x) => `categories:'${encodeURIComponent(x)}'`
              )
            )
          }

          if (selectedVersions.value.length > 0) {
            const versionFacets = []
            for (const facet of selectedVersions.value) {
              versionFacets.push('versions:' + facet)
            }
            formattedFacets.push(versionFacets)
          }

          if (onlyOpenSource.value) {
            formattedFacets.push(['open_source:true'])
          }

          if (selectedEnvironments.value.length > 0) {
            let environmentFacets = []

            const includesClient = selectedEnvironments.value.includes('client')
            const includesServer = selectedEnvironments.value.includes('server')
            if (includesClient && includesServer) {
              environmentFacets = [['client_side:required'], ['server_side:required']]
            } else {
              if (includesClient) {
                environmentFacets = [
                  ['client_side:optional', 'client_side:required'],
                  ['server_side:optional', 'server_side:unsupported'],
                ]
              }
              if (includesServer) {
                environmentFacets = [
                  ['client_side:optional', 'client_side:unsupported'],
                  ['server_side:optional', 'server_side:required'],
                ]
              }
            }

            formattedFacets = [...formattedFacets, ...environmentFacets]
          }

          if (projectType.value) {
            formattedFacets.push([`project_type:${projectType.value.actual}`])
          }

          params.push(`facets=${JSON.stringify(formattedFacets)}`)
        }

        const offset = (currentPage.value - 1) * maxResults.value
        if (currentPage.value !== 1) {
          params.push(`offset=${offset}`)
        }

        let url = 'search'

        if (params.length > 0) {
          for (let i = 0; i < params.length; i++) {
            url += i === 0 ? `?${params[i]}` : `&${params[i]}`
          }
        }

        return `${base}${url}`
      },
      {
        transform(hits) {
          noLoad.value = false
          return hits
        },
      }
    )

    const results = shallowRef(toRaw(rawResults))
    const pageCount = computed(() =>
      results.value ? Math.ceil(results.value.total_hits / results.value.limit) : 1
    )

    const onSearchChange = (newPageNumber) => {
      noLoad.value = true

      currentPage.value = newPageNumber

      if (query.value === null) {
        return
      }

      refreshSearch()

      if (process.client) {
        const router = useRouter()
        const obj = getSearchUrl((currentPage.value - 1) * maxResults.value, true)
        router.replace({ path: route.path, query: obj })
      }
    }

    const getSearchUrl = (offset, useObj) => {
      const queryItems = []
      const obj = {}

      if (query.value) {
        queryItems.push(`q=${encodeURIComponent(query.value)}`)
        obj.q = query.value
      }
      if (offset > 0) {
        queryItems.push(`o=${offset}`)
        obj.offset = offset
      }
      if (facets.value.length > 0) {
        queryItems.push(`f=${encodeURIComponent(facets.value)}`)
        obj.f = facets.value
      }
      if (orFacets.value.length > 0) {
        queryItems.push(`g=${encodeURIComponent(orFacets.value)}`)
        obj.g = orFacets.value
      }
      if (selectedVersions.value.length > 0) {
        queryItems.push(`v=${encodeURIComponent(selectedVersions.value)}`)
        obj.v = selectedVersions.value
      }
      if (onlyOpenSource.value) {
        queryItems.push('l=true')
        obj.l = true
      }
      if (showSnapshots.value) {
        queryItems.push('h=true')
        obj.h = true
      }
      if (selectedEnvironments.value.length > 0) {
        queryItems.push(`e=${encodeURIComponent(selectedEnvironments.value)}`)
        obj.e = selectedEnvironments.value
      }
      if (sortType.value.name !== 'relevance') {
        queryItems.push(`s=${encodeURIComponent(sortType.value.name)}`)
        obj.s = sortType.value.name
      }
      if (maxResults.value !== 20) {
        queryItems.push(`m=${encodeURIComponent(maxResults.value)}`)
        obj.m = maxResults.value
      }

      let url = `${route.path}`

      if (queryItems.length > 0) {
        url += `?${queryItems[0]}`

        for (let i = 1; i < queryItems.length; i++) {
          url += `&${queryItems[i]}`
        }
      }

      return useObj ? obj : url
    }

    return {
      query,
      results,
      facets,
      orFacets,
      selectedVersions,
      onlyOpenSource,
      showSnapshots,
      selectedEnvironments,
      sortTypes,
      sortType,
      maxResults,
      currentPage,
      pageCount,
      projectType,
      onSearchChange,
      getSearchUrl,
      searchLoading,
      noLoad,
    }
  },
  computed: {
    categoriesMap() {
      const categories = {}

      for (const category of this.$sortedCategories) {
        if (categories[category.header]) {
          categories[category.header].push(category)
        } else {
          categories[category.header] = [category]
        }
      }

      const newVals = Object.keys(categories).reduce((obj, key) => {
        obj[key] = categories[key]
        return obj
      }, {})

      return newVals
    },
  },
  methods: {
    clearFilters() {
      for (const facet of [...this.facets]) {
        this.toggleFacet(facet, true)
      }
      for (const facet of [...this.orFacets]) {
        this.toggleOrFacet(facet, true)
      }

      this.onlyOpenSource = false
      this.selectedVersions = []
      this.selectedEnvironments = []
      this.onSearchChange(1)
    },
    toggleFacet(elementName, doNotSendRequest = false) {
      const index = this.facets.indexOf(elementName)

      if (index !== -1) {
        this.facets.splice(index, 1)
      } else {
        this.facets.push(elementName)
      }

      if (!doNotSendRequest) {
        this.onSearchChange(1)
      }
    },
    toggleOrFacet(elementName, doNotSendRequest) {
      const index = this.orFacets.indexOf(elementName)
      if (index !== -1) {
        this.orFacets.splice(index, 1)
      } else {
        if (elementName === 'categories:purpur') {
          if (!this.orFacets.includes('categories:paper')) {
            this.orFacets.push('categories:paper')
          }
          if (!this.orFacets.includes('categories:spigot')) {
            this.orFacets.push('categories:spigot')
          }
          if (!this.orFacets.includes('categories:bukkit')) {
            this.orFacets.push('categories:bukkit')
          }
        } else if (elementName === 'categories:paper') {
          if (!this.orFacets.includes('categories:spigot')) {
            this.orFacets.push('categories:spigot')
          }
          if (!this.orFacets.includes('categories:bukkit')) {
            this.orFacets.push('categories:bukkit')
          }
        } else if (elementName === 'categories:spigot') {
          if (!this.orFacets.includes('categories:bukkit')) {
            this.orFacets.push('categories:bukkit')
          }
        } else if (elementName === 'categories:waterfall') {
          if (!this.orFacets.includes('categories:bungeecord')) {
            this.orFacets.push('categories:bungeecord')
          }
        }
        this.orFacets.push(elementName)
      }

      if (!doNotSendRequest) {
        this.onSearchChange(1)
      }
    },
    toggleEnv(environment, sendRequest) {
      const index = this.selectedEnvironments.indexOf(environment)
      if (index !== -1) {
        this.selectedEnvironments.splice(index, 1)
      } else {
        this.selectedEnvironments.push(environment)
      }

      if (!sendRequest) {
        this.onSearchChange(1)
      }
    },
    onSearchChangeToTop(newPageNumber) {
      if (process.client) {
        window.scrollTo({ top: 0, behavior: 'smooth' })
      }

      this.onSearchChange(newPageNumber)
    },
    onMaxResultsChange(newPageNumber) {
      newPageNumber = Math.max(
        1,
        Math.min(
          Math.floor(newPageNumber / (this.maxResults / this.previousMaxResults)),
          this.pageCount
        )
      )
      this.previousMaxResults = this.maxResults
      this.onSearchChange(newPageNumber)
    },
    cycleSearchDisplayMode() {
      this.$cosmetics.searchDisplayMode[this.projectType.id] = this.$cycleValue(
        this.$cosmetics.searchDisplayMode[this.projectType.id],
        this.$tag.projectViewModes
      )
      saveCosmetics()
      this.setClosestMaxResults()
    },
    setClosestMaxResults() {
      const view = this.$cosmetics.searchDisplayMode[this.projectType.id]
      const maxResultsOptions = this.maxResultsForView[view] ?? [20]
      const currentMax = this.maxResults
      if (!maxResultsOptions.includes(currentMax)) {
        this.maxResults = maxResultsOptions.reduce(function (prev, curr) {
          return Math.abs(curr - currentMax) <= Math.abs(prev - currentMax) ? curr : prev
        })
      }
    },
  },
})
</script>

<style lang="scss" scoped>
.normal-page__content {
  // Passthrough children as grid items on mobile
  display: contents;

  @media screen and (min-width: 1024px) {
    display: block;
  }
}

// Move the filters "sidebar" on mobile underneath the search card
.normal-page__sidebar {
  grid-row: 3;

  // Hide on mobile unless open
  @media screen and (max-width: 1024px) {
    display: none;

    &.open {
      display: block;
    }
  }
}

.filters-card {
  padding: var(--spacing-card-md);

  @media screen and (min-width: 1024px) {
    padding: var(--spacing-card-lg);
  }
}

.sidebar-menu {
  display: none;
}

.sidebar-menu_open {
  display: block;
}

.sidebar-menu-heading {
  margin: 1.5rem 0 0.5rem 0;
}

// EthicalAds
.content-wrapper {
  grid-row: 1;
}

.search-controls {
  display: flex;
  flex-direction: row;
  gap: var(--spacing-card-md);
  flex-wrap: wrap;
  padding: var(--spacing-card-md);
  grid-row: 2;

  .search-filter-container {
    display: flex;
    width: 100%;
    align-items: center;

    .sidebar-menu-close-button {
      max-height: none;
      // match height of the search field
      height: 40px;
      transition: box-shadow 0.1s ease-in-out;
      margin-right: var(--spacing-card-md);

      &.open {
        color: var(--color-button-text-active);
        background-color: var(--color-brand-highlight);
        box-shadow: inset 0 0 0 transparent, 0 0 0 2px var(--color-brand);
      }
    }

    .iconified-input {
      flex: 1;

      input {
        width: 100%;
        margin: 0;
      }
    }
  }

  .sort-controls {
    width: 100%;
    display: flex;
    flex-direction: row;
    gap: var(--spacing-card-md);
    flex-wrap: wrap;
    align-items: center;

    .labeled-control {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.5rem;

      .labeled-control__label {
        white-space: nowrap;
      }
    }

    .square-button {
      margin-top: auto;
      // match height of search dropdowns
      height: 40px;
      width: 40px; // make it square!
    }
  }
}

.search-controls__sorting {
  min-width: 14rem;
}

.labeled-control__label,
.labeled-control__control {
  display: block;
}

.pagination-before {
  grid-row: 4;
}

.search-results-container {
  grid-row: 5;
}

.pagination-after {
  grid-row: 6;
}

.no-results {
  text-align: center;
  display: flow-root;
}

.loading-logo {
  margin: 2rem;
}

#search-results {
  min-height: 20vh;
}

@media screen and (min-width: 750px) {
  .search-controls {
    flex-wrap: nowrap;
    flex-direction: row;
  }

  .sort-controls {
    min-width: fit-content;
    max-width: fit-content;
    flex-wrap: nowrap;
  }

  .labeled-control {
    align-items: center;
    display: flex;
    flex-direction: column !important;
    flex-wrap: wrap;
    gap: 0.5rem;
    max-width: fit-content;
  }

  .labeled-control__label {
    flex-shrink: 0;
    margin-bottom: 0 !important;
  }
}

@media screen and (min-width: 860px) {
  .labeled-control {
    flex-wrap: nowrap !important;
    flex-direction: row !important;
  }
}

@media screen and (min-width: 1024px) {
  .sidebar-menu {
    display: block;
    margin-top: 0;
  }

  .sidebar-menu-close-button {
    display: none;
  }

  .labeled-control {
    flex-wrap: wrap !important;
    flex-direction: column !important;
  }
}

@media screen and (min-width: 1100px) {
  .labeled-control {
    flex-wrap: nowrap !important;
    flex-direction: row !important;
  }
}

.server-banner {
  background: url(https://i.imgur.com/ZJSMEpt.png) left center;
  padding: var(--spacing-card-lg) 4rem var(--spacing-card-lg) 16rem;
  border-radius: var(--size-rounded-card);
  margin-bottom: var(--spacing-card-md);
  border: 3px solid var(--color-brand);
  display: flex;
  align-items: center;
  gap: 3rem;

  h2 {
    margin: 0;
    color: white;
  }

  .iconified-button {
    flex-shrink: 0;
  }
}
</style>
