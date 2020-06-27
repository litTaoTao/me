<template>
	<mescroll-vue
		ref="mescroll"
		:down="mescrollDown"
		:up="mescrollUp"
		:container="container"
		@init="mescrollInit">
		<div id="noDate">
			<slot name='listInfo'></slot>		<!-- 列表 -->
		</div>
	</mescroll-vue>
</template>
<script>
	import MescrollVue from './Mescroll.vue';
	export default {
		props:{
			pages:{
				type:Object,
				default: ()=>{
					return {
						num:0,
						size:15
					}
				}
			},
			//下拉参数配置
			scrollDown:{
				type:Object,
				default: ()=>{
					return {
						use:false,		//是否使用下拉
						isLock:true, //是否锁定下拉
					}
				}
			},
			//容器
			container:{
				type:String,	//默认为Mescroll.vue组件  可填写body
				default:""
			}
		},
		data() {
			return {
				// 下拉上拉数据开始
				mescroll: null,// mescroll实例对象
				mescrollDown:Object.assign({
					onMoving:this.meOnMoving,
					beforeLoading:this.meBeforeLoading
				},this.scrollDown),
				mescrollUp: {// 上拉加载的配置.
					callback: this.upCallback,// 上拉回调,此处简写; 相当于 callback: function(page, mescroll) { }
					//以下是一些常用的配置,当然不写也可以的.
					page: {
						num:this.pages.num,
						size:this.pages.size
					},
					htmlLoading: '<p class="upwarp-progress mescroll-rotate"></p><p class="upwarp-tip">加载中..</p>', //上拉加载中的布局
					htmlNodata: '<p class="upwarp-nodata" style="padding: 10px">-- 无更多数据 --</p>',
					empty: {
						//列表第一页无任何数据时,显示的空提示布局; 需配置warpId才显示
						warpId: "noDate", //父布局的id (1.3.5版本支持传入dom元素)
						icon: "", //图标,默认null,支持网络图
						tip: "无符合申请报案信息,请先进行报案" //提示
					},
					noMoreSize: 5, //如果列表已无数据,可设置列表的总数量要大于5才显示无更多数据;避免列表数据过少(比如只有一条数据),显示无更多数据会不好看
				},
			};
		},
		methods:{
			//下拉初始化
			mescrollInit (mescroll) {
				if(mescroll.isScrollBody){
					document.body.style.height="auto";
					this.$refs.mescroll.$refs.mescroll.style="inherit";
				}else{
					document.body.style.height="100%";
					this.$refs.mescroll.$refs.mescroll.style="absolute";
				}
				this.mescroll = mescroll;
			},
			//下拉
			upCallback (page,mescroll) {
				this.$emit("getDate",page,mescroll)
			},
			meBeforeLoading(mescroll , downwarp){
				if(mescroll.downHight>135){
					this.$emit("showPage",mescroll);
					//重置下拉距离
					mescroll.downwarp.style.height=0+"px";
					mescroll.downHight=0;
					return true;
				}
				return false;
			},
			meOnMoving(mescroll, rate, downHight){
				if(downHight>135){
					mescroll.downTipDom.innerHTML="松手有惊喜";
				}
			}
		},
		components:{
			MescrollVue
		}
	}
</script>

<style scoped="scoped">
	.mescroll {
		top:0;
		padding-bottom: 49px;
		box-sizing: border-box;
	}
</style>