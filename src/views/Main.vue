<style lang="less">
    @import "./main.less";
</style>
<template>
    <div class="main" :class="{'main-hide-text': shrink}">
        <!-- 侧边栏导航 -->
        <div class="sidebar-menu-con" :style="{width: shrink?'60px':'200px', overflow: shrink ? 'visible' : 'auto'}">
            <shrinkable-menu 
                :shrink="shrink"
                @on-change="handleSubmenuChange"
                :theme="menuTheme" 
                :before-push="beforePush"
                :open-names="openedSubmenuArr"
                :menu-list="menuList">
                <div style="cursor:pointer" slot="top" class="logo-con" @click="$router.push('/')">
                    <img v-show="!shrink"  src="../images/logo.png" key="max-logo" />
                    <img v-show="shrink" src="../images/logo-min.png" key="min-logo" />
                </div>
            </shrinkable-menu>
        </div>
        <div class="main-header-con" :style="{paddingLeft: shrink?'60px':'200px'}">
            <div class="main-header">
                <div class="navicon-con">
                    <Button :style="{transform: 'rotateZ(' + (this.shrink ? '-90' : '0') + 'deg)'}" type="text" @click="toggleClick">
                        <Icon type="navicon" size="32"></Icon>
                    </Button>
                </div>
                <div class="header-middle-con">
                    <div class="main-breadcrumb">
                        <breadcrumb-nav :currentPath="currentPath"></breadcrumb-nav>
                    </div>
                </div>
                <div class="header-avator-con">
                    <full-screen v-model="isFullScreen" @on-change="fullscreenChange"></full-screen>
                    <lock-screen></lock-screen>
                    <!-- <message-tip v-model="mesCount"></message-tip> -->
                    <theme-switch></theme-switch>
                    
                    <div class="user-dropdown-menu-con">
                        <Row type="flex" justify="end" align="middle" class="user-dropdown-innercon">
                            <Dropdown transfer trigger="click" @on-click="handleClickUserDropdown">
                                <a href="javascript:void(0)">
                                    <span class="main-user-name">{{ userName }}</span>
                                    <Icon type="arrow-down-b"></Icon>
                                </a>
                                <DropdownMenu slot="list">
                                    <DropdownItem name="ownSpace">个人中心</DropdownItem>
                                    <DropdownItem name="loginout" divided>退出登录</DropdownItem>
                                </DropdownMenu>
                            </Dropdown>
                            <Avatar :src="avatorPath" style="margin-left: 10px;"></Avatar>
                        </Row>
                    </div>
                </div>
            </div>
            <div class="tags-con">
                <tags-page-opened :pageTagsList="pageTagsList"></tags-page-opened>
            </div>
        </div>
        <div class="single-page-con" :style="{left: shrink?'60px':'200px'}">
            <div class="single-page">
                <keep-alive :include="cachePage">
                    <router-view></router-view>
                </keep-alive>
            </div>
        </div>
    </div>
</template>
<script>
    import shrinkableMenu from './main-components/shrinkable-menu/shrinkable-menu.vue';
    import tagsPageOpened from './main-components/tags-page-opened.vue';
    import breadcrumbNav from './main-components/breadcrumb-nav.vue';
    import fullScreen from './main-components/fullscreen.vue';
    import lockScreen from './main-components/lockscreen/lockscreen.vue';
    import messageTip from './main-components/message-tip.vue';
    import themeSwitch from './main-components/theme-switch/theme-switch.vue';
    import Cookies from 'js-cookie';
    import util from '@/libs/util.js';
    import {mapGetters, mapActions} from 'vuex';
    
    export default {
        components: {
            shrinkableMenu,
            tagsPageOpened,
            breadcrumbNav,
            fullScreen,
            lockScreen,
            messageTip,
            themeSwitch
        },
        data () {
            return {
                shrink: false,
                userName: '',
                isFullScreen: false,
                openedSubmenuArr: this.$store.state.app.openedSubmenuArr
            };
        },
        computed: {
            menuList () {
                return this.$store.state.app.menuList;
            },
            pageTagsList () {
                return this.$store.state.app.pageOpenedList;  // 打开的页面的页面对象
            },
            currentPath () {
                return this.$store.state.app.currentPath;  // 当前面包屑数组
            },
            avatorPath () {
                return require('./../images/asset.png');
            },
            cachePage () {
                return this.$store.state.app.cachePage;
            },
            lang () {
                return this.$store.state.app.lang;
            },
            menuTheme () {
                return this.$store.state.app.menuTheme;
            },
            mesCount () {
                return this.$store.state.app.messageCount;
            }
        },
        methods: {
            ...mapActions([
                'FollowProblePage',
                'saveFollowWay',
                'saveAccessUse',
                'saveFollowTemplate',
                'saveAccessBusines',
                'saveRoleManageRecord'
            ]),
            init () {
                let pathArr = util.setCurrentPath(this, this.$route.name);
                this.$store.commit('updateMenulist');
                if (pathArr.length >= 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
                this.userName = Cookies.get('user');
                let messageCount = 3;
                this.messageCount = messageCount.toString();
                this.checkTag(this.$route.name);
                this.$store.commit('setMessageCount', 3);
            },
            toggleClick () {
                this.shrink = !this.shrink;
            },
            handleClickUserDropdown (name) {
                if (name === 'ownSpace') {
                    util.openNewPage(this, 'ownspace_index');
                    this.$router.push({
                        name: 'ownspace_index'
                    });
                } else if (name === 'loginout') {
                    // 退出登录
                    this.$store.commit('logout', this);
                    this.$store.commit('clearOpenedSubmenu');
                    this.$router.push({
                        name: 'login'
                    });
                }
            },
            checkTag (name) {
                let openpageHasTag = this.pageTagsList.some(item => {
                    if (item.name === name) {
                        return true;
                    }
                });
                if (!openpageHasTag) {  //  解决关闭当前标签后再点击回退按钮会退到当前页时没有标签的问题
                    util.openNewPage(this, name, this.$route.params || {}, this.$route.query || {});
                }
            },
            handleSubmenuChange (val) {
                // console.log(val)
            },
            beforePush (name) {
                // if (name === 'accesstest_index') {
                /* return false; */
                // } else {
                //     return true;
                // }
                return true;
            },
            fullscreenChange (isFullScreen) {
                // console.log(isFullScreen);
            }
        },
        watch: {
            '$route' (to) {
                this.$store.commit('setCurrentPageName', to.name);
                let pathArr = util.setCurrentPath(this, to.name);
                if (pathArr.length > 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
                this.checkTag(to.name);
                localStorage.currentPageName = to.name;

                //清空缓存  再次进入页面清空 随访问题缓存的内容
                if(to.name!='voice'&&to.name!='followProblems'){
                    this.FollowProblePage({
                        title:'',
                        diseaseList:[],
                        followProble:{
                            followProblePage:1
                        }
                    });
                }


                //清空随访方案  再次进入页面清空 随访问题缓存的内容
                if(to.name!='way'&&to.name!='followWay'){
                    this.saveFollowWay({
                        name:'',
                        page:1,
                        departmentId:'',
                        diseaseId:"",
                        diseaseList:[]
                    });
                };

                //清空随访模板 再次进入页面清空 随访问题缓存的内容
                if(to.name!='template'&&to.name!='followTemplate'){
                    this.saveFollowTemplate({
                        name:'',
                        page:1,
                        diseaseId:'',
                        diseaseList:[]
                    });
                };



                ///清空用户列表 再次进入页面清空 随访问题缓存的内容
                if(to.name!='user_manage'&&to.name!='user_add'){
                    this.saveAccessUse({
                        page: 1,//当前页码
                        dpId: "",//科室Id
                        yhName: "", //用户名
                        types: "-1",//角色（0管理员，1医生）
                        state: "-1" //状态（0锁定，1正常）
                    });
                };
               
                //清空企业管理 再次进入页面清空 随访问题缓存的内容
                if(to.name!='access_business'&&to.name!='business_add'){
                    this.saveAccessBusines({
                        page:1,
                        searchResult:'',
                    });
                };

                //清空角色管理 再次进入页面清空 角色管理缓存的内容
                if(to.name!='role_manage'&&to.name!='role_add'){
                    this.saveRoleManageRecord({
                        page:1,
                        content:'',
                    });
                };







            },
            lang () {
                util.setCurrentPath(this, this.$route.name);  // 在切换语言时用于刷新面包屑
            }
        },
        mounted () {
            this.init();
        },
        created () {
            // 显示打开的页面的列表
            this.$store.commit('setOpenedList');
        }
    };
</script>
