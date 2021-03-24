<template>
    <ul class="sidenav app-sidenav" :class="{open: value}">
        <router-link 
            v-for="link in links"
            :key="link.title"
            v-slot="{ isActive }"
            active-class="active"
            :to="link.url"
            :exact="link.exact">
            <li
                :class="isActive && 'active'"
                >
                <a :href="link.url" class="waves-effect waves-grey pointer" >{{ link.title }}</a>
            </li>
        </router-link>
    </ul>
</template>

<script>
import LocalizeFilter from '@/filters/localize.filter'

export default {
    props: [ 'value' ],
    data: () => ({
        links: [
            {title: LocalizeFilter('Bill'),url: '/', exact: true},
            {title: LocalizeFilter('History'),url: '/history'},
            {title: LocalizeFilter('Planning'),url: '/planning'},
            {title: LocalizeFilter('NewRecord'),url: '/record'},
            {title: LocalizeFilter('Categories'),url: '/categories'}
        ]
    })
}
</script>