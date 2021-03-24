<template>
    <div>
            <div class="page-title">
                <h3>{{ 'Planning' | localize }}</h3>
                <h4>{{ info.bill | currency('RUB') }}</h4>
            </div>
            <Loader v-if="loading"/>
            <p v-else-if="!categories.length" class="center">Записей нет. Создать можно <router-link to="/record">тут</router-link> </p>
            <section v-else>
                <div v-for="cat in categories" :key="cat.id">
                    <p>
                        <strong>{{ cat.title }}:</strong>
                        {{ cat.spend | currency }} {{ 'Of' | localize }} {{ cat.limit | currency }} 
                    </p>
                    <div class="progress" v-tooltip.noloc="cat.tooltip">
                        <div 
                            class="determinate"
                            :class="[cat.progressColor]" 
                            :style="{width: cat.progressPercent + '%'}"></div>
                    </div>
                </div>
            </section>
        </div>
        
</template>

<script>
import {mapGetters} from 'vuex'
import currencyFilter from '@/filters/currency.filter'
import localizeFilter from '@/filters/localize.filter'

export default {
    metaInfo() {
        return {
            title: this.$title('Planning')
        }
    },
    name: 'planing',
    data: () => ({
        categories: [],
        loading: true
    }),
    computed: {
        ...mapGetters(['info'])
    },
    async mounted(){
        const records = await this.$store.dispatch('featchRecords')
        const categories = await this.$store.dispatch('featchCategories')
        this.categories = categories.map( cat => {
            const spend = records
                .filter(r => r.categoryId === cat.id)
                .filter(r => r.type === 'outcome')
                .reduce((total, record) => { // почитать об этом свойстве
                    return total += +record.amount
                    
                }, 0)
            const percent = 100 * spend / cat.limit
            const progressPercent = percent > 100 ? 100 : percent
            const progressColor = percent < 60 
                ? 'green'
                : percent < 100
                    ? 'yellow'
                    : 'red'
            const tooltipValue = cat.limit - spend
            const tooltip = `${tooltipValue < 0 ? localizeFilter('MoreThan') : localizeFilter('Stayed')} ${currencyFilter(Math.abs(tooltipValue))}`
            return {
                ...cat,
                progressPercent,
                progressColor,
                spend,
                tooltip
            }
                
        })
        this.loading = false
    }
    
}
</script>