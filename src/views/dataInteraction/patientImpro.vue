<template>
	<div>
		<div class="demo-upload-list" v-for="item in uploadList">
			<template v-if="item.status === 'finished'">
				<img :src="item.url">
				<div class="demo-upload-list-cover">
					<Icon type="ios-eye-outline" @click.native="handleView(item.name)"></Icon>
					<Icon type="ios-trash-outline" @click.native="handleRemove(item)"></Icon>
				</div>
			</template>
			<template v-else>
				<Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
			</template>
		</div>
		<Upload ref="upload" 
				:show-upload-list="false" 
				:on-success="handleSuccess" 
				:format="['xml','xls','png']" 
				:on-format-error="handleFormatError" 
				:on-exceeded-size="handleMaxSize" 
				:before-upload="handleBeforeUpload" 
				multiple 
				name="upfile"
				type="drag" 
				:action="API.Data.PatImport"
				style="display: inline-block;width:158px;">
			<div style="width: 158px;height:158px;display:flex;flex-direction:column;align-items:center;justify-content:center">
				<Icon type="document-text" size="50"></Icon>
				<br>
				点击导入病患信息
			</div>
		</Upload>
		<Button style="display:block;margin-top:20px;" type="primary"><a style="color:#fff" href="/assets/templatedoc/Patient.xls">下载患者模板</a></Button>
		<Modal title="上传错误" v-model="errorMsg">
			<Table border :columns="config" :data="dataList" ></Table>
			<Button style="display:block;margin-top:20px;" type="primary"><a style="color:#fff" href="/assets/templatedoc/患者信息错误.xls">下载患者错误信息</a></Button>
		</Modal>
	</div>
</template>
<script>
import { API } from '@/services';
export default {
	data() {
		return {
			API:API,
			defaultList: [

			],
			imgName: '',
			dataList:[],
			errorMsg:false,
			config: [
				{
					title: '错误行',
					key: 'rowNum',
					align: 'center',
					width:100
				},
				{
					title: '错误列',
					key: 'lieNum',
					align: 'center',
					width:100
				},
				{
					title: '错误信息',
					key: 'errMsg',
					align: 'center',
				},
			],
			visible: false,
			uploadList: []
		}
	},
	methods: {
		handleView(name) {
			this.imgName = name;
			this.visible = true;
		},
		handleRemove(file) {
			const fileList = this.$refs.upload.fileList;
			this.$refs.upload.fileList.splice(fileList.indexOf(file), 1);
		},
		handleSuccess(res, file) {
			res=JSON.parse(res);
			if(res.code==0){
				this.$Message.success("上传成功");
			}else{
				if(res.data){
					if(res.data.length>0){
						this.errorMsg=true;
						this.dataList=res.data;
						this.$Message.warning("上传失败");
					}else{
						this.$Message.warning("上传失败");	
					}
				}else{
					this.$Message.warning("上传失败");
				}
			}
			
		},
		handleFormatError(file) {
			this.$Notice.warning({
				title: '文件格式错误',
				desc:  file.name + ' 格式不正确, 请选择 xls 或者 xlsx 格式文件.'
			});
		},
		handleMaxSize(file) {
			this.$Notice.warning({
				title: 'Exceeding file size limit',
				desc: 'File  ' + file.name + ' is too large, no more than 2M.'
			});
		},
		handleBeforeUpload() {
			const check = this.uploadList.length < 5;
			if (!check) {
				this.$Notice.warning({
					title: 'Up to five pictures can be uploaded.'
				});
			}
			return check;
		}
	},
	mounted() {
		this.uploadList = this.$refs.upload.fileList;
	}
}
</script>
<style>
.demo-upload-list {
	display: inline-block;
	width: 60px;
	height: 60px;
	text-align: center;
	line-height: 60px;
	border: 1px solid transparent;
	border-radius: 4px;
	overflow: hidden;
	background: #fff;
	position: relative;
	box-shadow: 0 1px 1px rgba(0, 0, 0, .2);
	margin-right: 4px;
}

.demo-upload-list img {
	width: 100%;
	height: 100%;
}

.demo-upload-list-cover {
	display: none;
	position: absolute;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	background: rgba(0, 0, 0, .6);
}

.demo-upload-list:hover .demo-upload-list-cover {
	display: block;
}

.demo-upload-list-cover i {
	color: #fff;
	font-size: 20px;
	cursor: pointer;
	margin: 0 2px;
}
</style>