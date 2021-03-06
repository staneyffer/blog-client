<template>
	<v-card class="elevation-6">
		<vm-back-top></vm-back-top>
		<v-card-title primary-title>
			<div class="headline mx-auto">
				<span>{{post.title}}</span>
			</div>
			<v-btn color="primary" fab small v-if="canEdit" @click="deletePost">
				<v-icon>delete</v-icon>
			</v-btn>
			<v-btn color="primary" fab small v-if="canEdit" :to="toEditUrl + post.id">
				<v-icon>edit</v-icon>
			</v-btn>
		</v-card-title>
		<v-divider></v-divider>
		<v-card-text v-show="true">
			<!--<div v-html="post.contentHtml"></div>-->
			<div class="my-note markdown-body">
				<!--v-note-wrapper v-note-panel v-note-show v-show-content-->
				<div class="v-note-panel shadow">
					<div class="v-note-show">
						<div v-html="post.contentHtml" class="v-show-content scroll-style">
						</div>
					</div>
				</div>
			</div>
		</v-card-text>
		<v-divider/>
		<v-card-actions>
			<!-- prevent: 提交事件不再重载页面 -->
			<a @click.prevent="toInfo(cateRouter, post.category.id)" style="text-decoration: none; color:black">
				<v-icon>note</v-icon>
				<span>{{post.category.name || null}}</span>
			</a>

			<span
					v-bind:class="{'hidden-sm-and-down': true}"
					v-for="(tag, i) in post.tags" :key="i">
				<a @click.prevent="toInfo(tagRouter, tag.id)" style="text-decoration: none; color:black">
					<v-icon>bookmark</v-icon>
					<span>{{tag.name || null}}</span>
				</a>
			</span>

			<span
					v-bind:class="{'hidden-sm-and-down': true}"
					v-for="(sec, index) in post.sections" :key="index">
				<a @click.prevent="toInfo(secRouter, sec.id)" style="text-decoration: none; color:black">
					<v-icon>view_column</v-icon>
					<span>{{sec.name || null}}</span>
				</a>
			</span>
			<v-spacer></v-spacer>
			<v-icon>date_range</v-icon>
			<span>{{post.createTime}}</span>
		</v-card-actions>
	</v-card>
</template>

<script>
	import hljsCss from '../libs/markdown/core/hljs/lang.hljs.css.js'
	import {mapGetters} from 'vuex'
	import postApi from '../api/post'

	import {loadLink, loadScript} from '../libs/markdown/core/extra-function'

	export default {
		name: "ShowPost",
		props: {
			codeStyle: {
				type: String,
				default() {
					return 'monokai';
				}
			},
			post: {
				type: Object,
				default() {
					return ''
				}
			}
		},
		data() {
			return {
				s_external_link: {
					markdown_css: function () {
						return 'https://cdnjs.cloudflare.com/ajax/libs/github-markdown-css/2.9.0/github-markdown.min.css';
					},
					hljs_js: function () {
						return 'https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/highlight.min.js';
					},
					hljs_lang: function (lang) {
						return 'https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/languages/' + lang + '.min.js';
					},
					hljs_css: function (css) {
						if (hljsCss[css]) {
							return 'https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/styles/' + css + '.min.css';
						}
						return '';
					},
					katex_js: function () {
						return 'https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.8.3/katex.min.js';
					},
					katex_css: function () {
						return 'https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.8.3/katex.min.css';
					}
				},
				toEditUrl: '/update-post/',
				cateRouter: 'catePosts',
				tagRouter: 'tagPosts',
				secRouter: 'secPosts'
			}
		},
		mounted() {
			var $vm = this;
			$vm.loadExternalLink('markdown_css', 'css');
			$vm.loadExternalLink('katex_css', 'css')
			$vm.loadExternalLink('katex_js', 'js')
			$vm.loadExternalLink('hljs_js', 'js')
			/*$vm.loadExternalLink('katex_js', 'js', function() {
				$vm.initLanguage();
				$vm.iRender();
			})
			$vm.loadExternalLink('hljs_js', 'js', function() {
				$vm.initLanguage();
				$vm.iRender();
			})*/
			$vm.codeStyleChange($vm.codeStyle, true)
			// $vm.checkAuthor()
		},
		methods: {
			loadExternalLink(name, type, callback) {
				if (typeof this.s_external_link[name] != 'function') {
					if (this.s_external_link[name] != false) {
						console.error('external_link.' + name, 'is not a function, if you want to disabled this error log, set external_link.' + name, 'to function or false');
					}
					return;
				}
				var _obj = {
					'css': loadLink,
					'js': loadScript,
				};
				if (_obj.hasOwnProperty(type)) {
					_obj[type](this.s_external_link[name](), callback);
				}
			},
			codeStyleChange(val, isInit) {
				isInit = isInit ? isInit : false;
				if (typeof this.s_external_link.hljs_css != 'function') {
					if (this.s_external_link.hljs_css != false)
						console.error('external_link.hljs_css is not a function, if you want to disabled this error log, set external_link.hljs_css to function or false');
					return;
				}
				var url = this.s_external_link.hljs_css(val);
				if (url.length === 0 && isInit) {
					console.warn('hljs color scheme', val, 'do not exist, loading default monokai');
					url = this.s_external_link.hljs_css('monokai')
				}
				if (url.length > 0) {
					loadLink(url)
				} else {
					console.warn('hljs color scheme', val, 'do not exist, hljs color scheme will not change');
				}
			},
			deletePost(){
				if(!confirm('确认删除该文章吗?')){
					return
				}
				postApi.deletePostById(this.post.id)
					.then(res => {
						this.$log.debug(res)
						this.$notify({
							group: 'post',
							title: '文章删除成功',
						})
						this.$router.push({
							path: '/'
						})
					})
					.catch(err => {
						this.$log.debug(err)
					})
			},
			toInfo(routerName, id){
				this.$router.push({
					name: routerName,
					params: {id: id}
				})
			},
		},
		computed: {
			...mapGetters({
				isLogin: 'isLogin',
				curUser: 'userInfo'
			}),
			canEdit(){
				if(this.post.user){
					return this.isLogin && (this.curUser.id === this.post.user.id)
				}else{
					return false
				}
			}
		},
		watch: {
			codeStyle: function (val) {
				this.codeStyleChange(val)
			}
		}
	}
</script>

<style scoped>
	.my-note {
		/*position: relative;*/
		min-height: 300px;
		min-width: 50px;
		display: flex;
		flex-direction: column;
		transition: all 0.3s linear 0s;
		background: #ffffff;
		z-index: 1500;
		text-align: left;
	}
</style>