<template>
	<div>
		<div class="shopcart">
			<div class="content" @click="toggleList">
				<div class="content-left">
					<div class="logo-wrapper" >
						<div class="logo" :class="{'highlight': totalPrice > 0}">
							<span class="icon-shopping_cart" :class="{'highlight': totalPrice > 0}"></span>
						</div>
						<div class="num" v-show="totalCount">{{totalCount}}</div>
					</div>
					<div class="price" :class="{'highlight': totalPrice > 0}">￥{{totalPrice}}</div>
					<div class="desc">另需配送费￥{{deliveryPrice}}元</div>
				</div>
				<div class="content-right" @click.stop.prevent="pay">
					<!-- stop 阻止事件冒泡 prevent 阻止默认事件 -->
					<div class="pay" :class="payClass">{{payDesc}}</div>
				</div>
			</div>
			<div class="ball-container">
				<!-- <transition-group name="drop">
				@before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter"
					<div v-for="(ball,index) in balls" v-show="ball.show" class="ball" :key="index">
						<div class="inner inner-hook">{{index}}</div>
					</div>
				</transition-group> -->
			</div>
			<transition name="fold">
				<div class="shopcart-list" v-show="listShow">
					<div class="list-header">
						<h1 class="title">购物车</h1>
						<span class="empty" @click="empty">清空</span>
					</div>
					<div class="list-content" ref="listContent">
						<ul>
							<li class="food border-1px" v-for="food in selectFoods">
								<span class="name">{{food.name}}</span>
								<div class="price">
									<span>￥{{food.price * food.count}}</span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartControl :food="food"></cartControl>
								</div>
							</li>
						</ul>
					</div>
				</div>
			</transition>
		</div>
		<transition name="fade-mask">
			<div class="list-mask" v-show="listShow" @click="hideList"></div>
		</transition>
	</div>	
</template>
<script>
	import cartControl from '@/components/cartControl/cartControl';
	import BScroll from 'better-scroll';

	export default{
		data() {
			return {
				balls: [{
					show: false
				},
				{
					show: false
				},
				{
					show: false
				},
				{
					show: false
				},
				{
					show: false
				}],
				dropBalls: [],
				fold: true
			}
		},
		props: {
			selectFoods: {
				type: Array,
				default() {
					return [{
						count: 1,
						price: 4
					}];
				}
			},
			deliveryPrice: {
				type: Number,
				default: 0
			},
			minPrice: {
				type: Number,
				default: 0
			}
		},
		computed: {
			totalPrice() {
				let total = 0;
				
				this.selectFoods.forEach((food) => {
					total += food.price * food.count;
				});

				return total;
			},
			totalCount() {
				let count = 0
				this.selectFoods.forEach((food) => {
					count += food.count;
				});
				return count;
			},
			payDesc() {
				if(this.totalPrice === 0){
					return `￥${this.minPrice}元起送`;
				}else if(this.totalPrice < this.minPrice){
					let diff = this.minPrice - this.totalPrice;
					return `还差${diff}元起送`;
				}else{
					return '去结算';
				}
			},
			payClass() {
				if(this.totalPrice < this.minPrice){
					return 'not-enough';
				}
				return 'enough';
			},
			listShow() {
				if(!this.totalCount) {
					this.fold = true;
					return false
				}
				let show = !this.fold;

				if(show) {
					this.$nextTick(() => {

						if(!this.scroll){
							//console.log(this.refs)
							this.scroll = new BScroll(this.$refs.listContent,{
								click:true
							})
						}else{
							this.scroll.refresh(); //重新渲染计算
						}
						
					})
				}

				return show;
			}
		},
		methods: {
			drop(el) {

				for(let i=0; i<this.balls.length; i++){
					let ball = this.balls[i];
					if(!ball.show){
						
						ball.show = true;
						ball.el = el;
						this.dropBalls.push(ball);
						
						return;
					}
				}
			},
			/*beforeEnter(el){
				let count = this.balls.length;

				while(count--){
					let ball = this.balls[count];
					if(ball.show){
						let rect = ball.el.getBoundingClientRect(); //会返回一个矩形对象
						let x = rect.left - 32;
						let y = -(window.innerHeight - rect.top - 22);
						//el.style.display = 'block';
						el.style.webkitTransform = `translate3d(0,${y}px,0)`;
						el.style.transform = `translate3d(0,${y}px,0)`;
						let inner = el.querySelectorAll('.inner-hook')[0];
						inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
						inner.style.transform = `translate3d(${x}px,0,0)`;
						
					}
				}
			},*/
			//enter(el,done) {
				/* eslint-disable no-unused-vars */
				/*let rf = el.offsetHeight; //主动触发浏览器重绘
				this.$nextTick(() => {
					el.style.webkitTransform = 'translate3d(0,0,0)';
					el.style.transform = 'translate3d(0,0,0)';
					let inner = el.querySelectorAll('.inner-hook')[0];
					inner.style.webkitTransform = 'translate3d(0,0,0)';
					inner.style.transform = 'translate3d(0,0,0)';
					done();
				})
			},
			afterEnter(el){
				let ball = this.dropBalls.shift();
				if(ball){
					ball.show = false;
					el.style.display = 'none';
				}
			},*/
			toggleList() {
				if(!this.totalCount){
					return;
				}
				this.fold = !this.fold;
			},
			empty() {

				this.selectFoods.forEach((food) => {
					food.count = 0;
				})
			},
			hideList() {
				this.fold = true;
			},
			pay() {
				if(this.totalPrice < this.minPrice){
					return;
				}
				window.alert(`支付${this.totalPrice}元`)
			}
		},
		components:{
			cartControl
		},
		updated() {
			//console.log(4)
		}

	}
</script>
<style lang="stylus">
	@import "../../conmmon/stylus/mixin";
	.shopcart
		position: fixed
		left: 0
		bottom: 0
		z-index: 50
		width: 100%
		height: 48px
		.content
			display: flex
			background-color: #141d27;
			height: 100%
			font-size: 0
			color: rgba(255,255,255,0.4)
			.content-left
				flex: 1
				.logo-wrapper
					display: inline-block
					position: relative
					top: -10px
					margin: 0 12px
					padding: 6px
					width: 56px
					height: 56px
					box-sizing: border-box
					vertical-align: top
					border-radius: 50%
					background-color: #141d27
					.logo
						width: 100%
						height: 100%
						border-radius: 50%
						text-align: center
						background-color: #2b343c
						&.highlight
							background-color: rgb(0,160,220)
						.icon-shopping_cart
							font-size: 24px
							color: #80858a
							line-height: 44px
							&.highlight
								color: #fff
					.num
						position: absolute
						top: 0
						right: 0
						width: 24px
						height: 16px
						line-height: 16px
						text-align: center
						border-radius: 16px
						font-size: 9px
						font-weight: 400
						color: #fff
						background-color: rgb(240,20,20)
						box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4)
				.price
					display: inline-block
					vertical-align: top
					margin-top: 12px
					line-height: 24px
					padding-right: 12px
					box-sizing: border-box
					border-right: 1px solid rgba(255,255,255,0.1)
					font-size: 16px
					font-weight: 700
					&.highlight
						color: #fff
				.desc
					display: inline-block
					line-height: 24px
					vertical-align: top
					margin: 12px 0 0 12px
					font-size: 10px
					font-weight: 500
			.content-right
				flex: 0 0 105px
				width: 105px
				.pay
					height: 48px
					line-height: 48px
					text-align: center
					font-size: 12px
					font-weight: 700
					background-color: #2b333b
					&.not-enough
						background-color: #2b333b
					&.enough
						background-color: #00b43c
						color: #fff
		.ball-container
			.ball
				position: fixed
				left: 32px
				bottom: 22px
				z-index: 200
				background: red
				&.drop-enter-active
					transition: all 5s cubic-bezier(0.49, -0.29, 0.75, 0.41)
					.inner
						width: 16px
						height: 16px
						border-radius: 50%
						background: rgb(0,160,220)
						transition: all 5s linear
		.shopcart-list
			position: absolute
			top: 0
			left: 0
			z-index: -1
			width: 100%
			transform: translate3d(0,-100%,0)
			background: #fff
			.list-header
				height: 40px
				line-height: 40px
				padding: 0 18px
				background-color: #f3f5f7
				border-bottom: 1px solid rgba(7,17,27,0.1)
				.title
					float: left
					font-size: 14px
					color: rgb(7,17,27)
				.empty
					float: right
					font-size: 12px
					color: rgb(0,160,220)
			.list-content
				padding: 0 18px
				max-height: 217px
				overflow: hidden
				.food
					position: relative
					padding: 12px 0
					box-sizing: border-box
					border-1px(rgba(7,17,27,0.1))
					.name
						line-height: 24px
						font-size: 14px
						color: rgb(7,17,27)
					.price
						position: absolute
						right: 90px
						bottom: 12px
						line-height: 24px
						font-size: 14px
						color: rgb(240,20,20)
						font-weight: 700
					.cartcontrol-wrapper
						position: absolute
						right: 0
						bottom: 6px

		.fold-enter-active,.fold-leave-active
			transition: all 0.5s
			transform: translate3d(0,-100%,0)
		.fold-enter,.fold-leave-active
			transform: translate3d(0,0,0)
		.fold-leave
			transform: translate3d(0,-100%,0)
		
	.list-mask
		position: fixed
		top: 0
		left: 0
		width: 100%
		height: 100%
		z-index: 40
		backdrop-filter: blur(10px)
		opacity: 1
		background: rgba(7,17,27,0.6)
		&.fade-mask-enter-active,&.fade-mask-leave-active
			transition: all 0.5
			opacity: 0
			background: rgba(7,17,27,0)
		&.fade-mask-enter,&.fade-mask-leave-active
			opacity: 1
			background: rgba(7,17,27,0.6)

			
	
			
</style>