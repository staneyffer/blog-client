<template>
	<div>
		<mavon-editor
				:value="mdValue"
				:codeStyle="codeStyle"
				@change="handleChange"
				@imgAdd="$imgAdd"
				ref="md"></mavon-editor>
	</div>
</template>

<script>
	import postApi from '../api/post'

	let OSS = require('ali-oss')

	export default {
		name: "MdEditor",
		props: {
			mdValue: {
				type: String,
				default: ''
			},
			codeStyle: {
				type: String,
				default: 'monokai'
			}
		},
		data(){
			return{
				mdData: '',
				signature: {},
				ossClient: {}
			}
		},
		mounted(){
			this.mdData = this.mdValue
			this.getSignature()
		},
		methods: {
			handleChange(val, html){
				this.change(val, html)
			},
			change(val, html){
				this.$emit('change', val, html)
			},
			$imgAdd(pos, $file){
				let newName = this.genImgName($file.name)
				this.ossClient.put(newName, $file)
					.then(res => {
						this.$log.debug(res)
						let httpsUrl = 'https' + res.url.substring(5, res.url.length)
						this.$refs.md.$img2Url(pos, httpsUrl);
					})
					.catch(err => {
						this.$log.debug(err)
					})

			},
			getSignature(){
				postApi.getSignature()
					.then(res => {
						this.signature = res.data.body
						this.ossClient = new OSS(this.signature)
					})
					.catch(err => {
						this.$log.error(err)
					})
			},
			genImgName(oldName){
				let suffixName = '.' + oldName.split('.').pop()	// 得到后缀名
				const rnd = (start, end) => {
					// 指定范围的随机整数
					return Math.floor(Math.random() * (end - start) + start);
				}
				let newName = (new Date().getTime()).toString() + rnd(1000, 10000).toString() + suffixName
				return newName
			}
		}
	}
</script>

<style scoped>

</style>