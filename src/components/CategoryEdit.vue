<template>
    <div>
        <div class="page-subtitle">
            <h4>{{ 'Edit' | localize }}</h4>
        </div>

        <form @submit.prevent="submitHandler">
            <div class="input-field">
                <select 
                    ref="select"
                    v-model="current"
                >
                    <option 
                        v-for="cat in categories" 
                        :key="cat.id"
                        :value="cat.id"
                    >{{ cat.title }}</option>
                </select>
                <label>{{ 'SelectCategory' | localize }}</label>
            </div>
            <div class="input-field">
                <input 
                    id="name" 
                    type="text"
                    v-model="title"
                    :class="{invalid: requiredField}"
                >
                <label for="name">{{ 'Title' | localize }}</label>
                <span 
                    class="helper-text invalid"
                    v-if="requiredField"
                >
                    Введите название категории
                </span>
            </div>

            <div class="input-field">
                <input 
                    id="limit" 
                    type="number"
                    v-model="limit"
                    :class="{invalid: minValueField}"
                >
                <label for="limit">{{ 'Amount' | localize }}</label>
                <span 
                    class="helper-text invalid"
                    v-if="minValueField"
                >
                     {{'Message_MinValue' | localize}} {{$v.limit.$params.minValue.min}}
                </span>
            </div>

            <button class="btn waves-effect waves-light" type="submit">
                 {{'ButtonUpdate' | localize}}
                <i class="material-icons right">refresh</i>
            </button>
        </form>
    </div>
</template>

<script>
import {required, minValue} from 'vuelidate/lib/validators'
import localizeFilter from '@/filters/localize.filter'

export default {
    props: ['categories'],
    data: () => ({
        select: null,
        title: '',
        limit: 100,
        current: null
    }),
    validations: {
        title: {required},
        limit: {minValue: minValue(100)}
    },
   
    watch: {
        current(catId){
            const {title, limit} = this.categories.find( c => c.id === catId)
            this.title = title
            this.limit = limit
        }
    },
    created() {
        const {title, limit, id} = this.categories[0]

        this.current = id
        this.title = title
        this.limit = limit
    },
    destroyed() {
        if(this.select && this.select.destroy) {
            this.select.destroy
        }
    },
    mounted() {
        this.select = M.FormSelect.init(this.$refs.select);
        M.updateTextFields()
    },
    methods: {
        async submitHandler() {
            if (this.$v.$invalid) {
                this.$v.$touch()
                return
            }
            try {
                const categoryData = {
                    id: this.current,
                    title: this.title,
                    limit: this.limit
                }
                await this.$store.dispatch('updateCategory', categoryData)

                this.$message(localizeFilter('Category_HasBeenUpdated'))
                this.$emit('update',categoryData)
            } catch (e) {}
            
        }
    },
    computed: {
        requiredField() {
            return this.$v.title.$dirty && !this.$v.title.required
        },
        minValueField() {
            return this.$v.limit.$dirty && !this.$v.limit.minValue
        }
    }
}
</script>