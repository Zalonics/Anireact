<script setup>
    import { ref, onMounted } from 'vue'
    import { client } from '../pocketbase'
    import { resultsPerPage } from '../config.js'
    import { useHelpers } from '@/composables/useHelpers.js'

    import Gallery from '../components/Gallery/Gallery.vue'
    import Gallery__Search from '../components/Gallery/Gallery__Search.vue'

    const props = defineProps({
        page: {
            type: String,
            required: false,
            default: '1',
        },
    })

    const { capitalize } = useHelpers()

    const currentReactions = ref()
    const currentTag = ref('')
    const initialReactions = ref()
    const totalPages = ref()

    onMounted(async () => {
        document.title = 'Anime Reactions - Home'
        try {
            const pageNumber = +props.page
            const data = await client.records.getList(
                'reactions',
                pageNumber,
                resultsPerPage,
                {
                    sort: `-created`,
                }
            )
            totalPages.value = data.totalPages
            initialReactions.value = data.items
        } catch (e) {
            console.log(e)
        }
    })

    /**
     * @param {String} tag
     */
    const filterReactions = async (tag) => {
        if (tag.length > 2) {
            tag = capitalize(tag)
            currentTag.value = tag
            const data = await client.records.getFullList('reactions', 100, {
                filter: `emotion~"${tag}"`,
                sort: `-created`,
            })
            console.log(data)

            currentReactions.value = data
            return
        }
        currentReactions.value = initialReactions.value
        currentTag.value = ''
    }
</script>

<template>
    <div class="homepage">
        <Gallery__Search
            @filterReactions="filterReactions"
            :reactions="currentReactions"
            :searchTerm="currentTag"
        />
        <Gallery
            @filterReactions="filterReactions"
            :reactions="currentReactions"
            :totalPages="totalPages"
        />
    </div>
</template>

<style lang="scss" scoped>
    .homepage {
        padding: 15px;
    }
</style>
