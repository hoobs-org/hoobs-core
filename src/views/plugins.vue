<!-------------------------------------------------------------------------------------------------
 | hoobs-core                                                                                     |
 | Copyright (C) 2020 HOOBS                                                                       |
 |                                                                                                |
 | This program is free software: you can redistribute it and/or modify                           |
 | it under the terms of the GNU General Public License as published by                           |
 | the Free Software Foundation, either version 3 of the License, or                              |
 | (at your option) any later version.                                                            |
 |                                                                                                |
 | This program is distributed in the hope that it will be useful,                                |
 | but WITHOUT ANY WARRANTY; without even the implied warranty of                                 |
 | MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the                                  |
 | GNU General Public License for more details.                                                   |
 |                                                                                                |
 | You should have received a copy of the GNU General Public License                              |
 | along with this program.  If not, see <http://www.gnu.org/licenses/>.                          |
 -------------------------------------------------------------------------------------------------->

<template>
    <div id="plugins">
        <div class="info">
            <router-link to="/plugins" class="active">{{ $t("installed_packages") }}</router-link>
            <router-link v-if="user.admin" v-on:click="clearSearch()" to="/plugins/search">{{ $t("search") }}</router-link>
        </div>
        <div v-if="loaded && installed.length === 0" class="content">
            <div class="empty mobile-hide">{{ $t("no_plugins") }}</div>
            <div class="mobile-show">
                <router-link v-if="user.admin" v-on:click="clearSearch()" to="/plugins/search" class="button button-primary">{{ $t("add") }}</router-link>
            </div>
        </div>
        <div v-else class="content">
            <div v-for="(plugin, index) in installed" :key="`plugin-${index}`">
                <plugin-list v-if="user.admin || plugin.scope === 'hoobs'" :plugin="plugin" />
            </div>
            <div class="mobile-show">
                <router-link v-if="user.admin" v-on:click="clearSearch()" to="/plugins/search" class="button button-primary">{{ $t("add") }}</router-link>
            </div>
        </div>
    </div>
</template>

<script>
    import PluginList from "@/components/plugin-list.vue";

    export default {
        name: "plugins",

        components: {
            "plugin-list": PluginList
        },

        data() {
            return {
                loaded: false,
                installed: []
            }
        },

        computed: {
            user() {
                return this.$store.state.user;
            }
        },

        async mounted() {
            this.installed = await this.api.get("/plugins");
            this.loaded = true;
        }
    }
</script>

<style scoped>
    #plugins {
        flex: 1;
        padding: 0;
        display: flex;
        overflow: hidden;
    }

    #plugins .info {
        width: 230px;
        padding: 20px 0 20px 20px;
        overflow: auto;
    }

    #plugins .info a,
    #plugins .info a:link,
    #plugins .info a:active,
    #plugins .info a:visited {
        padding: 10px;
        border-bottom: 1px var(--border) solid;
        color: var(--text);
        text-decoration: none !important;
        display: block;
        cursor: pointer;
    }

    #plugins .info a:hover {
        color: var(--text-dark);
    }

    #plugins .info .active {
        font-weight: bold;
        color: var(--title-text) !important;
    }

    #plugins .content {
        flex: 1;
        padding: 20px 20px 0 20px;
        display: flex;
        flex-direction: column;
        overflow: auto;
    }

    #plugins .empty {
        width: 90%;
        padding: 20px;
        text-align: center;
    }

    @media (min-width: 300px) and (max-width: 815px) {
        #plugins .info {
            display: none;
        }
    }
</style>
