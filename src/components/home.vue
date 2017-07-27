<template>
    <div class="post-list">
        <!-- Overlay //-->
        <transition name="overlay-show">
            <div class="overlay" v-if="show" v-on:click="closePost()">
            </div>
        </transition>
        <!-- Filters //-->
        <div class="filters" v-bind:class="{ 'filter-active': filterActive }">
            <div class="container pad">
                <div class="filter name-filter">
                    <h4 class="small-heading">Filter by Name</h4>
                    <input type="text" placeholder="Enter search term" name="" v-model="nameFilter">
                </div>
                <div class="filter">
                    <h4 class="small-heading">Filter by Category</h4>
                    <div class="radio-wrap">
                        <input type="radio" value="" v-model="categoryFilter">
                        <label>All</label>
                    </div>
                    <div class="radio-wrap" v-for="category in categories" v-if="category.name != 'Uncategorised'">
                        <input type="radio" v-bind:value="category.id" v-model="categoryFilter">
                        <label>{{ category.name }}</label>
                    </div>
                </div>
            </div>
        </div>
        <!-- Filters Button //-->
        <div class="container pad filter-revealer">
            <a href="#" class="btn orange-btn" v-on:click="filterVisibility" v-bind:class="{ 'filter-active': filterActive }">{{ filterActive ? 'Close Filters' : 'Open Filters' }}</a>
        </div>
        <!-- Posts //-->
        <div class="container post-list">
            <article v-for="post in filteredPosts" class="post">
                <a v-on:click="getThePost(post.id)">
                    <img v-bind:src="post.fi_300xx180" alt="">
                </a>
                <div class="post-content">
                    <h2 class="small-heading">{{ post.title.rendered }}</h2>
                    <span class="btns">
                        <span class="btn" v-for="category in post.cats">
                            {{ category.name }}
                        </span>
                    </span>
                </div>
            </article>
        </div>
        <!-- Preview Panel //-->
        <transition name="preview-panel">
            <div class="single-preview" v-if="show">
                <h2 v-html="post[0].excerpt.rendered"></h2>
                <img v-bind:src="post[0].full" alt="">
                <span v-html="post[0].excerpt.rendered"></span>
                <router-link class="btn orange-btn" to="{name:'post', params:{postSlug: post[0].slug}}">About</router-link>
                <a v-on:click="getThePost(post[0].prev_post)" v-if="post[0].prev_post" class="prev"></a>
                <a v-on:click="getThePost(post[0].next_post)" v-if="post[0].next_post" class="next"></a>
                <button class="close" v-on:click="closePost()">&#215;</button>
            </div>
        </transition>
    </div>
</template>

<script>
export default {
    name: 'PostList',
    data() {
        console.log('data')
        return {
            slug: 'posts',
            posts: [],
            post: '',
            pages: [],
            page: '',
            nameFilter: '',
            categoryFilter: '',
            categories: [],
            filterActive: false,
            show: false
        }
    },
    created() {
        console.log('created')
        this.$http.get('https://restapi.li1.home-trial.com/wp-json/wp/v2/posts?per_page=20').then(response => {
            this.posts = response.body
        }, response => {
            // error callback
        })
        this.$http.get('https://restapi.li1.home-trial.com/wp-json/wp/v2/categories').then(response => {
            this.categories = response.body
        }, response => {
            // error callback
        })
        this.$http.get(`https://restapi.li1.home-trial.com/wp-json/wp/v2/pages?slug=$(posts)`).then(response => {
            this.pages = response.body
            document.title = this.pages[0]._yoast_wpseo_title[0]
            document.head.querySelector('meta[name=description]').content = this.pages[0]._yoast_wpseo_metadesc[0]
        }, response => {
            // error callback
        })
    },
    methods: {
        getThePost(id) {
            var posts = this.posts
            this.show = true
            function filterPosts(el) {
                return el.id === id
            }
            this.post = posts.filter(filterPosts)
        },
        filterVisibility() {
            if (this.filterActive) {
                this.filterActive = false
            } else {
                this.filterActive = true
            }
        },
        closePost() {
            this.show = false
        }
    },
    computed: {
        filteredPosts() {
            return this.posts.filter((post) => {
                const toLower = new RegExp(this.nameFilter, 'i')
                const rendered = post.title.rendered.match(toLower)
                const postsInCats = post.categories.indexOf(this.categoryFilter) >= 0
                if (postsInCats) {
                    return rendered
                } else if (this.categoryFilter === '') {
                    return rendered
                }
            })
        }
    }
}
</script>

<style>

</style>
