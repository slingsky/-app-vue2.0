<template>
	<div class="goods">
		<div class="menu-wrapper" ref="menuWrapper">
			<ul>
				<li v-for="(val,index) in goods" class="menu-item" :class="{'current': currentIndex == index}" @click="selectMenu(index,$event)">
					<span class="text border-1px">
						<span v-show="val.type>0" class="icon" :class="classMap[val.type]"></span>{{val.name}}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foodsWrapper">
			<ul>
				<li v-for="item in goods" class="food-list food-list-hook">
					<h1 class="title">{{item.name}}</h1>
					<ul>
						<li v-for="food in item.foods" class="food-item border-1px">
							<div class="icon">
								<img :src="food.icon" alt="" width="100%" />
							</div>
							<div class="content">
								<h2 class="name">{{food.name}}</h2>
								<p class="desc">{{food.description}}</p>
								<div class="extra">
									<span class="count">月售{{food.sellCount}}份</span>
									<span>好评率{{food.rating}}%</span>
								</div>
								<div class="price">
									<span class="now">￥{{food.price}}</span><span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartControl :food="food"  @cartAdd="_drop"></cartControl>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>
		<shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" ref="shopcart"></shopcart>
	</div>
</template>
<script type="text/javascript">
	import BScroll from 'better-scroll';
	import shopcart from '@/components/shopcar/shopcar';
	import cartControl from '@/components/cartControl/cartControl';

	const ERR_OK = 0;
	export default{
		props: {
			seller: {
				type: Object
			}
		},
		data() {
			return {
				goods: [],
				classMap: '',
				listHeight: [],
				scrollY: 0
			}
		},
		mounted() {
			this.classMap = ['decrease','discount','special','invoice','guarantee'];
			this.$http.get('/api/goods').then(res => {
				let msg = res.body;
				if(msg.errno === ERR_OK){
					this.goods = msg.data;
					this.$nextTick(() => {
						this._initScroll();
						this._calculateHeight()
					})
				}
			})
		},
		methods: {
			selectMenu(index,event) {
				//原生派发没有event._constructed
				if(!event._constructed){
					return;
				}

				let foodList = this.$refs.foodsWrapper.querySelectorAll('.food-list-hook');
				let el = foodList[index];
				this.foodsScroll.scrollToElement(el,300);

			},
			_initScroll() {
				this.meunScroll = new BScroll(this.$refs.menuWrapper,{
					click: true
				});
				this.foodsScroll = new BScroll(this.$refs.foodsWrapper,{
					click: true,
					probeType: 3 //监测滚动位置
				});

				//监听滚动返回y值
				this.foodsScroll.on('scroll', (pos) => {
					this.scrollY = Math.abs(Math.round(pos.y));
				});

			},
			_calculateHeight() {
				let foodList = this.$refs.foodsWrapper.querySelectorAll('.food-list-hook');
				let height = 0;

				this.listHeight.push(height);

				foodList.forEach((val,index) => {
					height += val.clientHeight;
					this.listHeight.push(height); //计算各个分类高度
				});
			},
			_drop(target) {
				this.$refs.shopcart.drop(target)
			}

		},
		computed: {
			currentIndex() {
				for(let i=0; i<this.listHeight.length; i++){
					let height1 = this.listHeight[i];
					let height2 = this.listHeight[i+1];
					if(!height2 || this.scrollY >= height1 && this.scrollY < height2){
						return i;
					} 
				}
				return 0;
			},
			selectFoods() {
				let foods = [];
				this.goods.forEach((goods) => {
					goods.foods.forEach((food) => {
						if(food.count){
							foods.push(food)
						}
					})
				})
				return foods;
			}
		},
		components: {
			shopcart,
			cartControl
		}
	}
</script>
<style lang="stylus">
	@import "../../conmmon/stylus/mixin";
	.goods
		display: flex
		position: absolute
		top: 174px
		bottom: 46px
		width: 100%
		overflow: hidden
		.menu-wrapper
			flex: 0 0 80px /* 等分 内容不足是否缩放 占位空间 */
			width: 80px  /* 不写在安卓浏览器会有问题 */
			background: #f3f5f7
			.menu-item
				display: table
				height: 50px
				width: 56px
				padding: 0 12px
				line-height: 14px
				&.current
					position: relative
					margin-top: -1px
					background: #fff
					z-index: 10
					font-weight: 700
					.text
						border-none()
				.icon
					display: inline-block
					width: 12px
					height: 12px
					vertical-align: top
					margin-right: 2px
					background-size: 12px 12px
					background-repeat: no-repeat
					&.decrease
						bg-image('decrease_3')
					&.discount
						bg-image('discount_3')
					&.guarantee
						bg-image('guarantee_3')
					&.invoice
						bg-image('invoice_3')
					&.special
						bg-image('special_3')
			.text
				display: table-cell
				width: 56px
				vertical-align: middle
				font-size: 12px
				border-1px(raba(7,17,27,0.1))

		.foods-wrapper
			flex: 1
			.title
				padding-left: 14px
				height: 26px
				line-height: 26px
				border-left: 2px solid #d9dde1
				font-size: 12px
				color: rgb(147,153,159)
				background: #f3f5f7
			.food-item
				display: flex
				margin: 18px
				padding-bottom: 18px
				border-1px(rgba(7,17,27,0.1))
				&:last-child
					border-none()
					margin-bottom: 0
				.icon
					flex: 0 0 50px
					margin-right: 10px
				.content
					flex: 1
					.name
						font-size: 14px
						height: 14px
						color: rgb(7,17,27)
						margin: 2px 0 8px 0
						line-height: 14px
					.desc,.extra
						line-height: 10px
						font-size: 10px
						color: rgb(147,153,159)
					.desc
						margin-bottom: 8px
						line-height: 12px
					.extra
						&.count
							margin-right: 12px
					.price
						font-weight: 700
						line-height: 24px
						.now
							margin-right: 18px
							font-size: 14px
							color: rgb(240,20,20)
						.old
							font-size: 10px
							text-decoration: line-through
							color: rgb(7,17,27)
					.cartcontrol-wrapper
						position: absolute
						right: 0
						bottom: 12px


</style>