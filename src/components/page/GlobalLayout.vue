<template>
    <el-container class="layout">
        <el-aside :width="menuProps.collapse ? '64px' : '240px'" class="layout-aside">
            <side-menu></side-menu>
        </el-aside>
        <el-container>
            <el-header class="layout-header border-b bg-white"
                       :style="{
                            marginLeft : menuProps.collapse ? '64px' : '240px',
                            background : headProps.theme[headProps.theme.default].backgroundColor,
                            color : headProps.theme[headProps.theme.default].textColor
                       }"
            >
                <global-header></global-header>
            </el-header>
            <el-main class="layout-main" :style="{marginLeft : menuProps.collapse ? '64px' : '240px'}">
                <router-view v-if="isRouterAlive"></router-view>
            </el-main>
        </el-container>
    </el-container>

</template>
<script>
    import {mapState, mapMutations} from 'vuex'
    import SideMenu from "@/components/menu/SideMenu";
    import GlobalHeader from "./GlobalHeader"
    import {localRead} from '@/utils/modules/tools'

    export default {
        name: 'GlobalLayout',
        components: {
            SideMenu, GlobalHeader,
        },
        provide() {
            return {
                reload: this.reload
            }
        },
        computed: {
            ...mapState({
                menuProps: ({app}) => app.menuProps,
                headProps: ({app}) => app.headProps
            })
        },
        data() {
            return {
                isRouterAlive : true
            }
        },
        watch: {
            '$route': {
                handler(to, from) {
                    this.handleNav(to.matched)
                },
                immediate: true
            }
        },
        methods: {
            ...mapMutations({
                setLang: 'SET_LANG',
                setActiveName: 'ACTIVE_NAME',
                setOpenNames: 'OPEN_NAMES',
                setActiveNavTag: 'ACTIVE_NAV_TAG',
                setActiveBreadCream: 'ACTIVE_BREAD_CREAM',
            }),
            reload() {
                this.isRouterAlive = false;
                this.$nextTick(() => {
                    this.isRouterAlive = true;
                })
            },
            handleNav(matched) {
                if (matched.length) {
                    const {path, parent, name, meta: {title, icon}} = matched[matched.length - 1]
                    const result = this.findOpenNames(parent)
                    const activeName = path
                    const openNames = result.map(item => item.path)
                    result.unshift({path, name, title, icon})
                    this.setActiveName(activeName)
                    this.setOpenNames(openNames)
                    this.setActiveNavTag({path, name, title,icon})
                    this.setActiveBreadCream(result.reverse())

                }
            },
            findOpenNames(obj, resultList = []) {
                if (obj && obj.constructor === Object) {
                    let {path, name, meta: {title, icon}, parent} = obj
                    resultList.push({path, name, title, icon})
                    this.findOpenNames(parent, resultList)
                }
                return resultList.filter(item => Boolean(item.path))
            }
        },
        mounted() {
            // 设置初始语言
            this.setLang(localRead('lang') || this.$i18n.locale)
        }
    }
</script>
<style scoped lang="less">
    .layout {
        &-aside {
            background-color: rgb(238, 241, 246);
            position: fixed;
            overflow: hidden;
            height: 100vh;
        }
        &-header {
            height: auto !important;
            padding: 0px;
        }
        &-main {
            padding: 0px;
            overflow: hidden;
        }
    }
</style>
