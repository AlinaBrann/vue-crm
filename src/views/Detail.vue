<template>
    <div>
        <Loader v-if="loading"/>
        <p v-else-if="!record" class="center">Записи с <b>id={{$route.params.id}}</b> не существует.</p>
        <div v-else>
            <div class="row">
                <div class="col s12">
                    <div class="breadcrumb-wrap">
                        <router-link to="/history" class="breadcrumb">{{ 'History' | localize }}</router-link>
                        <a @click.prevent class="breadcrumb">
                            {{ record.typeText }}
                        </a>
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="col s12 m6">
                    <div 
                        class="card" 
                        :class="[record.typeClass]">
                        <div class="card-content">
                            <p>{{ 'Description' | localize }}: {{ record.description }}</p>
                            <p>{{ 'Amount' | localize }}: {{ record.amount | currency }}</p>
                            <p>{{ 'Category' | localize }}: {{ record.categoryName }}</p>

                            <small>{{ record.date | date('datatime') }}</small>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import LocalizeFilter from '@/filters/localize.filter'

export default {
    metaInfo() {
        return {
            title: this.$title('Detail_Title')
        }
    },
    name: 'detail',
    data: () => ({
        loading: true,
        record: [],
    }),
    async mounted() {
        const id = this.$route.params.id
        const record = await this.$store.dispatch('featchRecordById', id)
        const category = await this.$store.dispatch('featchCategoryById', record.categoryId)
        this.record = {
            ...record,
            categoryName: category.title,
            typeText: record.type === 'income' ? LocalizeFilter('Income') : LocalizeFilter('Outcome'),
            typeClass: record.type === 'income' ? 'green lighten-4' : 'red lighten-4',
        }
        this.loading = false
    }
}
</script>