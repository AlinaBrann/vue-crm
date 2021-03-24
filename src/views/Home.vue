<template>
	<div class="home">
		<div>
			<div class="row">
				<div class="col s12">
					<div class="page-title">
						<h3>{{ 'Bill' | localize }}</h3>

						<button class="btn waves-effect waves-light btn-small" @click="refresh">
							<i class="material-icons">refresh</i>
						</button>
					</div>
				</div>
			</div>
			

			<Loader v-if="loading"/>

			<div v-else class="row">
				<HomeBill
					:rates="currency.rates"
				/>

				<HomeCurrency
					:rates="currency.rates"
					:date="currency.date"
				/>
			</div>
		</div>
	</div>
</template>

<script>
	import HomeBill from '@/components/HomeBill'
	import HomeCurrency from '@/components/HomeCurrency'
	import Loader from '../components/app/Loader.vue'
	import LocalizeFilter from '@/filters/localize.filter'


	export default {
		metaInfo() {
			return {
				title: LocalizeFilter('Bill')
			}
		},
		name: 'Home',
		data: () => ({
			loading: true,
			currency: null
		}),
		async mounted(){
			this.currency = await this.$store.dispatch('fetchCurrency')
			this.loading = false
		},
		methods: {
			async refresh(){
				this.loading = true	
				this.currency = await this.$store.dispatch('fetchCurrency')
				this.loading = false
			}
		},
		components: {
			HomeBill,
			HomeCurrency,
			Loader
		}
	}
</script>