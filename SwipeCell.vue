<template>
	<div
		class="cell_container"
		v-click-outside="handleClickOutside"
		@click="getClickHandler('cell')">
		<div
			:style="{'transform':
			'translateX('+(offset+(isElastic?elasticX:0))+'px)','transition-duration':dragging?'0s':'0.6s'}">
			<!-- <div ref="cellLeft" class="cell_left" @click="getClickHandler('left', true)">
				<div>收藏</div>
				<div>添加</div>
			</div> -->
			<div
				@touchend="onClick"
				class="cell_content">SwipeCell</div>
			<div ref="cellRight"
				class="cell_right"
				@click="getClickHandler('right', true)">
				<div
					:class="type?'divPostion':''"
					ref="remove"
					:style="{'background':'#ccc','padding-left':'10px','padding-right':10+(isElastic?Math.abs(elasticX/3):0)+'px','transition-duration':dragging?'0s':'0.6s'}">标记</div>
				<div 
					:class="type?'divPostion':''" 
					ref="tag" 
					:style="{'transform': type?'translateX('+(-offset*removeWidth/cellRightWidth-(isElastic?elasticX/3:0))+'px)':'','padding-left':'10px','padding-right':10+(isElastic?Math.abs(elasticX/3):0)+'px','transition-duration':dragging?'0s':'0.6s','background':'#000'}">不再关注</div>
				<div 
					:class="type?'divPostion':''" 
					:style="{'transform': type?'translateX('+(-offset*(removeWidth+tagWidth)/cellRightWidth-(isElastic?elasticX/3*2:0))+'px)':'','padding-left':'10px','padding-right':10+(isElastic?Math.abs(elasticX/3):0)+'px','transition-duration':dragging?'0s':'0.6s'}">删除</div>
			</div>
		</div>
	</div>
</template>
<script>
import ClickOutside from 'vue-click-outside';
import { TouchMixin } from '@/components/mixins/touch';
export default{
	name:"SwipeCell",
	props: {
		// @deprecated
		// should be removed in next major version, use beforeClose instead
		onClose: Function,
		disabled: Boolean,
		leftWidth: [Number, String],
		rightWidth: [Number, String],
		beforeClose: Function,
		stopPropagation: Boolean,
		name: {
			type: [Number, String],
			default: '',
		},
		//
		type:{
			type:[Number,String],
			default:1 //0 常规   1 定位
		},
		isElastic:{  //弹性
			type:Boolean,
			default:true
		}
	},
	data(){
		return {
			offset: 0,
			dragging: true,
			//-位移
			elasticX:0,
			removeWidth:0,
			tagWidth:0,
			cellRightWidth:0,
			cellLeftWidth:0
		}
	},
	computed: {
		computedLeftWidth() {
			return +this.leftWidth || this.getWidthByRef('cellLeft');
		},

		computedRightWidth() {
			return +this.rightWidth || this.getWidthByRef('cellRight');
		},
	},
	mounted() {
		//防止弹性效果影响宽度
		this.cellRightWidth = this.getWidthByRef('cellRight');
		this.cellLeftWidth = this.getWidthByRef('cellLeft');
		this.removeWidth = this.getWidthByRef('remove');
		this.tagWidth = this.getWidthByRef('tag');
		this.bindTouchEvent(this.$el);
	},
	mixins: [
		TouchMixin
	],
	directives: {
		ClickOutside
	},
	methods: {
		getWidthByRef(ref) {
			if (this.$refs[ref]) {
				const rect = this.$refs[ref].getBoundingClientRect();
				//type=1定位时获取宽度为0，为此采用获取子元素宽度之和
				if(!rect.width){
					let childWidth = 0;
					for(const item of this.$refs[ref].children){
						childWidth += item.getBoundingClientRect().width
					}
					return childWidth;
				}
				return rect.width;
			}
			return 0;
		},

		handleClickOutside(e){
			if(this.opened) this.close()
		},

		// @exposed-api
		open(position) {
			const offset =
			position === 'left' ? this.computedLeftWidth : -this.computedRightWidth;

			this.opened = true;
			this.offset = offset;

			this.$emit('open', {
				position,
				name: this.name,
				// @deprecated
				// should be removed in next major version
				detail: this.name,
			});
		},

		// @exposed-api
		close(position) {
			this.offset = 0;

			if (this.opened) {
				this.opened = false;
				this.$emit('close', {
					position,
					name: this.name,
				});
			}
		},

		onTouchStart(event) {
			if (this.disabled) {
				return;
			}
			this.startOffset = this.offset;
			this.touchStart(event);
		},

		range(num, min, max) {
			return Math.min(Math.max(num, min), max);
		},

		preventDefault(event, isStopPropagation) {
			/* istanbul ignore else */
			if (typeof event.cancelable !== 'boolean' || event.cancelable) {
				event.preventDefault();
			}

			if (this.isStopPropagations) {
				stopPropagation(event);
			}
		},

		stopPropagations(event) {
			event.stopPropagation();
		},

		onTouchMove(event) {
			if (this.disabled) {
				return;
			}

			this.touchMove(event);
			if (this.direction === 'horizontal') {
				this.dragging = true;
				this.lockClick = true;
				const isPrevent = !this.opened || this.deltaX * this.startOffset < 0;
				if (isPrevent) {
					this.preventDefault(event, this.stopPropagation);
				}
				
				this.offset = this.range(
					this.deltaX + this.startOffset,
					-this.computedRightWidth,
					this.computedLeftWidth
				);
				//增加弹性
				if(this.computedRightWidth && this.offset === -this.computedRightWidth || this.computedLeftWidth && this.offset === this.computedLeftWidth){
					//
					this.preventDefault(event, this.stopPropagation);
					//弹性系数
					this.elasticX = (this.deltaX + this.startOffset - this.offset)/4;
				}
			}
		},

		onTouchEnd() {
			if (this.disabled) {
				return;
			}
			//回弹
			this.elasticX = 0
			if (this.dragging) {
				this.toggle(this.offset > 0 ? 'left' : 'right');
				this.dragging = false;
				// compatible with desktop scenario
				setTimeout(() => {
					this.lockClick = false;
				}, 0);
			}
		},

		toggle(direction) {
			const offset = Math.abs(this.offset);
			const THRESHOLD = 0.15;
			const threshold = this.opened ? 1 - THRESHOLD : THRESHOLD;
			const { computedLeftWidth, computedRightWidth } = this;

			if (
			computedRightWidth &&
			direction === 'right' &&
			offset > computedRightWidth * threshold
			) {
				this.open('right');
			} else if (
			computedLeftWidth &&
			direction === 'left' &&
			offset > computedLeftWidth * threshold
			) {
				this.open('left');
			} else {
				this.close();
			}
		},

		onClick(position = 'outside') {
			this.$emit('click', position);

			if (this.opened && !this.lockClick) {
				if (this.beforeClose) {
					this.beforeClose({
						position,
						name: this.name,
						instance: this,
					});
				} else if (this.onClose) {
					this.onClose(position, this, { name: this.name });
				} else {
					this.close(position);
				}
			}
		},

		getClickHandler(position, stop) {
			return (event) => {
				if (stop) {
					event.stopPropagation();
				}
				this.onClick(position);
			};
		},
	}
}
</script>
<style lang="stylus" scoped>
.cell_container{
	position: relative;
	overflow: hidden;
	height:48px;
	div{
		height: 100%;
		.cell_content{
			height: 100%;
			width: 100%;
			text-align: center;
			&:active{
				background: #e8e8e8;
			}
		}
		.cell_left,.cell_right{
			position: absolute;
			top: 0;
			height: 100%;
			display: flex;
			color: #fff;
			.divPostion{
				position: absolute;
			}
			div{
				white-space:nowrap;
				display: flex;
				align-items: center;
				background: #ccc;
			}
		}
		.cell_left{
			left: 0;
			transform:translateX(-100%);
		}
		.cell_right{
			right: 0;
			transform:translateX(100%);
		}
	}
}
</style>