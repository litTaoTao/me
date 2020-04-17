<template>
	<mescroll-vue
		ref="mescroll"
		:up="mescrollUp"
		@init="mescrollInit">
		<div id="noDate">
			<slot name='listInfo'></slot>		<!-- 列表 -->
		</div>
	</mescroll-vue>
</template>
<script>
	import MescrollVue from './Mescroll.vue';
	export default {
		data() {
			return {
				// 下拉上拉数据开始
				mescroll: null,// mescroll实例对象
				mescrollUp: {// 上拉加载的配置.
					callback: this.upCallback,// 上拉回调,此处简写; 相当于 callback: function(page, mescroll) { }
					//以下是一些常用的配置,当然不写也可以的.
					page: {
						num:0,
						size:5        //每页显示待传
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
				this.mescroll = mescroll;
			},
			//下拉
			upCallback (page,mescroll) {
				this.$emit("getDate",page,mescroll)
			}
		},
		components:{
			MescrollVue
		}
	}
</script>

<style scoped="scoped">
	.mescroll {
		position: absolute;
		top:0;
		padding-bottom: 49px;
		box-sizing: border-box;
	}
</style>
