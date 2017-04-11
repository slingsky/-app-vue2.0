<template>
	<div class="cartControl">
		<transition name="move">
			<div class="cart-decrease" @click="decreaseCart($event)" v-show="food.count > 0">
					<span class="inner icon-remove_circle_outline"></span>
			</div>
		</transition>
		<div class="cart-count" v-show="food.count > 0">{{food.count}}</div>
		<div class="cart-add icon-add_circle" @click="addCart($event)"></div>
	</div>
</template>
<script type="text/javascript">
	import Vue from 'vue';

	export default{
		props: {
			food: {
				type: Object
			}
		},
		methods: {
			addCart(event) {
				if(!event._constructed){
					return;
				}
				
				if(!this.food.count) {
					Vue.set(this.food,'count',1);
				}else{
					this.food.count ++;
				}

				this.$emit('cartAdd', event.target);

			},
			decreaseCart(event) {
				if(!event._constructed){
					return;
				}
				
				if(this.food.count > 0) {
					this.food.count --;
				}
			}
		},
		mounted() {
		
		}
	}
</script>
<style lang="stylus">
	.cartControl
		font-size: 0
		.cart-decrease
			display: inline-block
			padding: 6px
			.inner
				display: inline-block
				font-size: 24px
				line-height: 24px
				color: rgb(0,16,220)
		.cart-count
			display: inline-block
			font-size: 10px
			width: 12px
			padding-top: 6px
			text-align: center
			vertical-align: top
			line-height: 24px
			color: rgb(147,153,159)
		.cart-add
			display: inline-block
			padding: 6px
			font-size: 24px
			line-height: 24px
			color: rgb(0,16,220)
		.move-enter-active,.move-leave-active
			transition: all 0.4s linear
			.inner
				transition: all 0.4s linear
				transform: rotate(0)
		.move-enter,.move-leave-active
			opacity: 0
			transform: translate3d(24px,0,0)
			.inner
				transform: rotate(180deg)
			
</style>