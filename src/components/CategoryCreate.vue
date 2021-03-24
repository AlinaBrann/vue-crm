<template>
    <div>
        <div class="page-subtitle">
            <h4>{{ 'Create' | localize }}</h4>
        </div>

        <form @submit.prevent="submitHandler">
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
                    {{ 'Message_MinValue' | localize }} {{$v.limit.$params.minValue.min}}
                </span>
            </div>

            <button class="btn waves-effect waves-light" type="submit">
                {{ 'Create' | localize }}
                <i class="material-icons right">add</i>
            </button>
        </form>
    </div>
</template>

<script>
import {required, minValue} from 'vuelidate/lib/validators'
import localizeFilter from '@/filters/localize.filter'

export default {
    data: () => ({
        title: '',
        limit: 100
    }),
    validations: {
        title: {required},
        limit: {minValue: minValue(100)}
    },
    mounted() {
        M.updateTextFields()
    },
    methods: {
        async submitHandler() {
            if (this.$v.$invalid) {
                this.$v.$touch()
                return
            }
            try {
                const category = await this.$store.dispatch('createCategory', {
                    title: this.title,
                    limit: this.limit
                })
                this.title = ''
                this.limit = 100
                this.$v.$reset()
                this.$message(localizeFilter('Category_HasBeenCreated'))
                this.$emit('created', category)
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