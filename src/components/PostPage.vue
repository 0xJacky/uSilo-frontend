<template>
	<div class="post-page">
		<div class="meta">
			<span>
				<a-icon type="eye"/>
				{{ post.visits }}
			</span>
			<span>
				<a-icon type="message"/>
				{{ post.comments_total }}
			</span>
		</div>
		<div class="post-content"><p>{{ post.content }}</p></div>
		<div class="post-img" v-if="post.upload_id">
			<img :src="post.upload.url"/>
		</div>

		<favour type="post" :id="post.id" :likes_-number="post.likes" :dislikes_-number="post.dislikes"/>

		<div class="post-info">
			<a-popconfirm
					v-if="is_login"
					title="你确定要删除这条动态?"
					@confirm="remove(post.id)" okText="是的"
					cancelText="再想想">
				<a href="javascript:;">删除</a> ·
			</a-popconfirm>
			<a-popconfirm
					title="你确定要举报这条动态?"
					@confirm="report(post.id)" okText="是的"
					cancelText="再想想">
				<a href="javascript:;">举报</a> ·
			</a-popconfirm>
			<span class="category"><router-link
					:to="'/category/'+post.category.id">{{ post.category.name }}</router-link>
			</span>
			·
			<span class="created-at">{{ moment(post.created_at).fromNow() }}</span>
		</div>
		<a-divider/>
		<p class="comment-label">评论</p>
		<editor :post_id="post.id"/>
		<comment-block
				v-for="root in post.comments.root"
				:key="root.id"
				:comment="root"
				:comments="post.comments.children"/>
	</div>
</template>

<script>
    import moment from 'moment'
    import CommentBlock from "./CommentBlock"
    import Editor from "./Editor"
    import Vue from 'vue'
    import {VueReCaptcha} from 'vue-recaptcha-v3'
    import Favour from "./Favour";

    Vue.use(VueReCaptcha, {
        siteKey: process.env.VUE_APP_RECAPTCHA_SITEKEY,
        loaderOptions: {
            useRecaptchaNet: true
        }
    })

    require('moment/locale/zh-cn')
    moment.locale('zh-cn')

    export default {
        name: "PostPage",
        components: {Favour, Editor, CommentBlock},
        props: {
            id: String
        },
        data() {
            return {
                moment,
                post: {
                    comments: {
                        root: [],
                        children: {}
                    },
                    category: {}
                }
            }
        },
        computed: {
            should_reload_comment() {
                return this.$store.getters.should_reload_comment
            },
            is_login() {
                return this.$store.getters.is_login
            }
        },
        watch: {
            id() {
                this.init()
            },
            should_reload_comment() {
                if (this.should_reload_comment) {
                    this.init()
                }
            },
        },
        created() {
            this.init()
        },
        methods: {
            init() {
                const t = this
                this.$http.get('/post?id=' + this.id)
                    .then((response) => {
                        this.post = response
                        this.$store.dispatch('comment_reloaded')
                    })
                    .catch(function (error) {
                        t.$message.error(error.error)
                    });
            },
            report(id) {
                this.$recaptchaLoaded().then(() => {
                    this.$recaptcha('/report').then((token) => {
                        const t = this
                        this.$http.post('/report', {post_id: id, token: token})
                            .then(() => {
                                this.$message.success('举报成功')
                            })
                            .catch(function (error) {
                                t.$message.error(error.error)
                            });
                    });
                });
            },
            remove(id) {
                this.$http.delete('/post', {data: {id: id}})
                    .then(() => {
                        this.$message.success('删除成功')
                        this.$emit('get_list')
                    })
                    .catch((error) => {
                        // eslint-disable-next-line no-console
                        console.log(error)
                    })
            }
        }
    }
</script>

<style scoped>
	.meta {
		text-align: right;
	}

	.meta span {
		margin: 5px;
	}

	.post-content {
		white-space: pre-wrap;
		margin: 0 0 10px 0;
		line-height: 20px;
	}

	.post-content p {
		font-size: 15px;
	}

	.post-img img {
		width: 50%;
	}


	@media (prefers-color-scheme: dark) {
		.post-content {
			color: #fff;
		}

		.comment-label {
			color: #fff;
		}

		.favour .action {
			color: #fff;
		}
	}
</style>
