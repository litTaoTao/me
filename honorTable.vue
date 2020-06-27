<template>
	<div class="honorTable" ref="honorTable">
		<transition name="fadeOut">
			<div class="tab" v-show="!isHidePage">
				<down-load ref='mescroll' :scrollDown="scrollDown" :container="container" :pages="pages" id="mescroll" @getDate="getDate" @showPage="showPage">
					<div slot='listInfo' ref='scroll'>
						<div class="box">
							<table @click="$router.push('/teamList')" cellpadding="0" v-for="(it,index) in listData" :key="index">
								<thead>
									<tr>
										<th colspan=3>您的业绩</th>
									</tr>
									<tr>
										<td>指标</td>
										<td>业绩</td>
										<td>入围标准</td>
									</tr>
								</thead>
								<tbody>
									<tr v-for="it in list">
										<td>指323sdfsdfsdfsd标</td>
										<td>业23绩</td>
										<td>入围23标准</td>
									</tr>
								</tbody>
							</table>
						</div>
						
					</div>	
				</down-load>
			</div>
		</transition>
		<!-- 隐藏页 -->
		<transition name="fadeUP">
			<div v-show="isHidePage" class="hidePage">
				<div class="hidePageContent">
					<div>
						<!-- 内容 -->
						<div v-for="i in 280">{{i}}</div>
					</div>
				</div>
				<div class="toHome" @click="showPage">返回首页</div>
			</div>
		</transition>
	</div>
</template>	
<script>
import DownLoad from '@/components/model/DownLoad';
import DatePicker from '@/components/model/DatePicker';
import Dialog from '@/components/model/Dialog';
export default{
	name:"HonorTable",
	props:{
		list:{			//列表数据
			type:Number,
			default:5
		},
		isLoad:{		//是否启用下拉刷新
			type:Boolean,
			default:false
		},
		container:{
			type:String,
			default:""
		},
		scrollDown:{
			type:Object,
			default: ()=>{
				return {
					use:true,
					isLock:false
				}
			}
		}
	},
	data(){
		return {
			date:"2019-9-9",
			isChange:false,
			listData:[],
			isVisible:false,
			listData:[],
			isHidePage:false,
			pages:{
				size:2,
				num:0
			},
		}
	},
	mounted(){
		if(!this.container){
			this.$refs.honorTable.style.height="100%"
		}
	},
	components:{DownLoad,DatePicker,Dialog},
	methods:{
		showPage(){
			this.isHidePage=!this.isHidePage;
		},
		processSearch(){
			console.log("您点击了日期")
			this.$emit("showDate")
		},
		searchData(){
			this.listData=[]
			this.$refs.mescroll.mescroll.resetUpScroll()
		},
		dialogClose(){
			this.isVisible=false;
		},
		dialogShow(){
			console.log(9999)
			//this.isVisible=true;
		},
		getDate(page,mescroll){
			try{
				let that=this;
				setTimeout(function(){
					var arr = [34,44,45]
			  		if (page.num === 1) that.listData = []
					that.listData = that.listData.concat(arr);
					that.$nextTick(()=>{
						mescroll.endSuccess(arr.length);
					})
				},1000)
	  	 		
	 		}
	 		catch(err){
	 			this.$nextTick(()=>{
					mescroll.endSuccess(0);
				})
	 		}
		},
		iptFocus(){
			this.isChange=true;
		},
		iptBlur(){
			this.isChange=false;
		}
	}
}
</script>
<style lang="stylus" scoped>
.hidePage{
	position: fixed;
	height: 100%;
	width: 100%;
	left: 0;
	background: yellow;
	z-index: 100;
	.hidePageContent{
		height: 100%;
    	overflow: auto;
    	width: 100%;
	}
	.toHome{
		position: absolute;
	    bottom: 50px;
	    left: 50%;
        background: #000;
	    opacity: 0.7;
	    padding: 10px 15px;
        color: #fff;
	    border-radius: 27px;
	    transform: translateX(-50%);
	}
}
.honorTable{
	box-sizing: border-box;
	display: flex;
	flex-direction: column;
	.ipt{
		&>.focusClass{
			background: #fff;
			color: #000;
			border: 1px solid #F1F1F1;
		}
		&>.blurClass{
			background: #F1F1F1;
			color: #B9B9B9;
		}
		&>input{
			width: 300px;
			height: 30px;
			line-height: 14px;
			border-radius: 5px;
			margin: 15px;
			margin-left: 0;
			font-size: 12px;
			padding-left: 15px;
    		box-sizing: border-box;
    		outline:none;
		}
	}
	.tab{
		position: relative;
		overflow-y: auto;
		flex: 1;
	}
	&>div{
		display: flex;
		justify-content: space-between;
		align-items: center;
		color: #656565;
		font-size: 13px;
		.date{
			font-size: 14px;
			&>span{
				color: #00a4a1;
				position: relative;
				padding-right: 20px;
				&>i{
					display: inline-block;
					border: 5px solid;
		    		border-color: #00a4a1 transparent transparent;
		    		position: absolute;
		    		top: 50%;
		    		transform: translateY(-25%);
		    		margin-left: 8px;
				}
			}
			
		}
	}
	.box{
		padding: 0 20px;
		table{
			width: 100%;
			border-radius: 5px;
			margin-top: 15px;
			td{
				border:1px solid #eee;
				text-align: center;
				padding: 10px 0;
			}
			th{
				background: #EDB10F;
				height: 35px;
				line-height: 35px;
				color: #fff;
				border-top-left-radius:8px;
				border-top-right-radius:8px;
			}
		}
	}
	
	>>>.container .dialog-main .dialog-box .dialog-content{
		background: #fff;
		padding: 0
	}
	>>>.container .dialog-main .dialog-box .dialog-btn{
		border: 0;
		height: 20px;
		width: 50px;
		line-height: 21px;
		margin: auto;
		background: #000;
		color: #FDE9D9;
		font-size: 12px;
		border-radius: 4px;
		letter-spacing: 2px;
	}
}

.fadeUP-enter {
	/*opacity: 0;*/
	transform: translateY(-100%);
}
.fadeUP-enter-active {
	transition: transform .5s;
}
.fadeUP-leave-to {
	transform: translateY(-100%);
}
.fadeUP-leave-active {
	transition: transform .5s;
}
.fadeOut-enter {
	/*opacity: 0;*/
	transform: translateY(100%);
}
.fadeOut-enter-active {
	transition: transform .5s;
}
.fadeOut-leave-to {
	transform: translateY(100%);
}
.fadeOut-leave-active {
	transition: transform .5s;
}

</style>