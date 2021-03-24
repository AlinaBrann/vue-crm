<template>
    <div>
        <div class="page-title">
            <h3>{{ 'NewRecord' | localize }}</h3>
        </div>
        
        <Loader v-if="loading"/>

        <p v-else-if="!categories.length" class="center">Категорий нет. Создать можно <router-link to="/categories">тут</router-link> </p>

        <form class="form" v-else @submit.prevent="submitHandler">
            <div class="input-field">
                <select ref="select" v-model="category">
                    <option 
                        v-for="cat in categories" 
                        :key="cat.id" 
                        :value="cat.id"
                    >
                        {{ cat.title }}
                    </option>
                </select>
                <label>{{'SelectCategory' | localize}}</label>
            </div>

            <p>
                <label>
                    <input 
                        class="with-gap" 
                        name="type" 
                        v-model="type"
                        type="radio" 
                        value="income" />
                    <span>{{ 'Income' | localize }}</span>
                </label>
            </p>

            <p>
                <label>
                    <input 
                        class="with-gap" 
                        name="type" 
                        v-model="type"
                        type="radio" 
                        value="outcome" />
                    <span>{{ 'Outcome' | localize }}</span>
                </label>
            </p>

            <div class="input-field">
                <input 
                    id="amount" 
                    v-model.number="amount"
                    :class="{invalid: minValueField}"
                    type="number">
                <label for="amount">{{ 'Amount' | localize }}</label>
                <span 
                    class="helper-text invalid"
                    v-if="minValueField"
                >
                    {{'Message_MinValue' | localize}} {{$v.amount.$params.minValue.min}}
                </span>
            </div>

            <div class="input-field">
                <input 
                    id="description" 
                    v-model="description"
                    :class="{invalid: requiredField}"
                    type="text">
                <label for="description">{{ 'Description' | localize }}</label>
                <span v-if="requiredField" class="helper-text invalid">{{ 'Message_EnterDescription' | localize }}</span>
            </div>

            <button class="btn waves-effect waves-light" type="submit">
                Создать
                <i class="material-icons right">send</i>
            </button>
        </form>
    </div>
</template>

<script>
import {required, minValue} from 'vuelidate/lib/validators'
import {mapGetters} from 'vuex'
import localizeFilter from '@/filters/localize.filter'

export default {
    metaInfo() {
        return {
            title: this.$title('NewRecord')
        }
    },
    name: 'record',
    data: () => ({
        loading: true,
        select: null,
        category: null,
        categories: [],
        type: 'outcome',
        amount: 1,
        description: ''
    }),
    async mounted() {
        this.categories = await this.$store.dispatch('featchCategories')
        this.loading = false
        setTimeout(() => {
            this.select = M.FormSelect.init(this.$refs.select);
            M.updateTextFields()
        },0)
        if(this.categories.length) {
            this.category = this.categories[0].id
        }
        
    },
    destroyed() {
        if(this.select && this.select.destroy) {
            this.select.destroy
        }
    },
    validations: {
        description: {required},
        amount: {minValue: minValue(1)}
    },
    computed: {
        requiredField() {
            return this.$v.description.$dirty && !this.$v.description.required
        },
        minValueField() {
            return this.$v.amount.$dirty && !this.$v.amount.minValue
        },
        ...mapGetters(['info']),
        canCreateRecord(){
            if(this.type === 'income') {
                return true
            }
            return this.info.bill >= this.amount
        }
    },
    methods: {
        async submitHandler() {
            if (this.$v.$invalid) {
                this.$v.$touch()
                return
            }
            if(this.canCreateRecord) {
                try {
                    await this.$store.dispatch('createRecord', {
                        categoryId: this.category,
                        amount: this.amount,
                        description: this.description,
                        type: this.type,
                        date: new Date().toJSON()
                    })
                    const bill = this.type === 'income'
                        ? this.info.bill + this.amount
                        : this.info.bill - this.amount
                    await this.$store.dispatch('updateInfo', {bill})
                    this.$message(localizeFilter('RecordHasBeenCreated'))
                    this.$v.$reset()
                    this.amount = 1
                    this.description = ''
                } catch (error) {
                    console.log('top');
                }
                
            } else {
                this.$message(`${localizeFilter('NotEnoughMoney')} (${this.amount - this.info.bill})`)
            }
        },
    },
}
</script>
