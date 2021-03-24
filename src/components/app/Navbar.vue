<template>
    <nav class="navbar grey darken-3">
        <div class="nav-wrapper">
            <div class="navbar-left">
                <a 
                    href="#"
                    @click.prevent="$emit('toggleSidebar')"
                >
                    <i class="material-icons white-text">dehaze</i>
                </a>
                <span class="white-text">{{ currentData | date('datetime') }}</span>
            </div>

            <ul class="right hide-on-small-and-down">
                <li>
                    <a 
                        ref="dropdown"
                        id='dropdowner'
                        class="dropdown-trigger white-text" 
                        href="#" 
                        data-target="dropdown">
                        {{ name }}
                        <i class="material-icons right">arrow_drop_down</i>
                    </a>

                    <ul id='dropdown' class='dropdown-content'>
                        <li>
                            <router-link to="/profile" class="black-text">
                                <i class="material-icons">account_circle</i>{{ 'ProfileTitle' | localize }}
                            </router-link>
                        </li>
                        <li class="divider" tabindex="-1"></li>
                        <li>
                            <a 
                                href="#" 
                                @click.prevent="logout"
                                class="black-text">
                                <i class="material-icons">assignment_return</i>{{ 'Logout' | localize }}
                            </a>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
    </nav>
</template>

<script>
export default {
    data: () => ({
        currentData: new Date(),
        interval: null,
        dropdown: null
    }),
    methods: {
        async logout() {
            await this.$store.dispatch('logout')
            this.$router.push('/login?message=logout')
        }
    },
    computed: {
        name() {
            return this.$store.getters.info.name
        }
    },
    mounted() {
        this.interval = setInterval(() => {
            this.currentData = new Date()
        }, 1000)
        const dd = document.getElementById('dropdowner');
        M.Dropdown.init(dd, {
            constrainWidth: true,
            alignment: 'right'
        })

       
    },
    beforeDestroy() {
        clearInterval(this.interval)
        if (this.dropdown && this.dropdown.destroy) {
            this.dropdown.destroy()
        }
    }
}
</script>