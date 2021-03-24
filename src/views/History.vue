<template>
    <div>
        <div class="page-title">
            <h3>{{ 'RecordHistory' | localize }}</h3>
        </div>

        <div class="history-chart">
            <canvas ref="canvas"></canvas>
        </div>

        <Loader v-if="loading"/>
        <p v-else-if="!records.length" class="center">Записей нет. Создать можно <router-link to="/record">тут</router-link> </p>
        <section v-else>
            <HistoryTable :records="items" />
        </section>
        <Paginate 
            v-model="page"
            :page-count="pageCount"
            :click-handler="pageChangeHandler"
            :prev-text="'Prev' | localize"
            :next-text="'Next' | localize"
            :container-class="'pagination'"
            :page-class="'waves-effect'"
        />
    </div>
</template>

<script>
import paginationMixin from '@/mixins/pagination.mixin'
import {Pie} from 'vue-chartjs'
import HistoryTable from '@/components/HistoryTable'
import LocalizeFilter from '@/filters/localize.filter'

export default {
    metaInfo() {
        return {
            title: this.$title('History')
        }
    },
    name: 'history',
    mixins: [paginationMixin],
    extends: Pie,
    data: () => ({
        loading: true,
        records: []
    }),
    async mounted() {
        this.records = await this.$store.dispatch('featchRecords')
        const categories = await this.$store.dispatch('featchCategories')
        this.setup(categories)

        this.renderChart({
            labels: categories.map(s => s.title),
            datasets: [{
                label: LocalizeFilter('CostsByCategory'),
                data: categories.map(c => {
                    return this.records.reduce((total, r) => {
                        if(r.categoryId === c.id && r.type === 'outcome') {
                            total += r.amount
                        }
                        return total
                    }, 0)
                }),
                backgroundColor: [
                    'rgba(255, 99, 132, 0.2)',
                    'rgba(54, 162, 235, 0.2)',
                    'rgba(255, 206, 86, 0.2)',
                    'rgba(75, 192, 192, 0.2)',
                    'rgba(153, 102, 255, 0.2)',
                    'rgba(255, 159, 64, 0.2)'
                ],
                borderColor: [
                    'rgba(255, 99, 132, 1)',
                    'rgba(54, 162, 235, 1)',
                    'rgba(255, 206, 86, 1)',
                    'rgba(75, 192, 192, 1)',
                    'rgba(153, 102, 255, 1)',
                    'rgba(255, 159, 64, 1)'
                ],
                borderWidth: 1
            }]
        })

        this.loading = false
    },
    methods: {
        setup(categories){
            this.setupPagination(this.records.map(record => {
                return {
                    ...record,
                    categoryName: categories.find(c => c.id === record.categoryId).title,
                    typeClass: record.type === 'income' ? 'green' : 'red',
                    typeText: record.type === 'income' ? LocalizeFilter('Income') : LocalizeFilter('Outcome')
                }
                
            }))
        }
    },
    components: {
        HistoryTable
    }
}
</script>