<template>
	<!--商品详情页-->
	<view v-if="goods_info.goods_name" class="goods-detail-container">
		<swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
			<swiper-item v-for="(item,i) in goods_info.pics" :key="i">
				<image :src="item.pics_big" @click="preview(i)"></image>
			</swiper-item>
		</swiper>
		
		<view class="goods-info-box">
			<view class="price">￥{{goods_info.goods_price}}</view>
			<view class="goods-info-body">
				<view class="goods-name">{{goods_info.goods_name}}</view>
				<view class="favi">
					<uni-icons type="star" size="18" color="gray"></uni-icons>
					<text>收藏</text>
				</view>
			</view>
			<view class="yf">快递：免运费</view>
		</view>
		
		<rich-text :nodes="goods_info.goods_introduce"></rich-text>
		<view class="goods_nav">
			<uni-goods-nav 
				:fill="true"  
				:options="options" 
				:buttonGroup="buttonGroup"  
				@click="onClick" 
				@buttonClick="buttonClick" 
		/>
		</view>
	</view>
</template>

<script>
	import {mapState,mapMutations,mapGetters} from 'vuex'
	
	export default {
		computed:{
			...mapState('m_cart',[]),
			...mapGetters('m_cart',['total'])//在vuex里拿数据
		},
		watch:{
			/* total(newVal){
				const findResult = this.options.find(x => x.text === '购物车')
				if(findResult){
					findResult.info = newVal
				}
			} */
			//watch监听器的普通用法，用immediate属性当值第一次绑定的时候立即执行
			total:{ //改造为对象的形式才能实现一加载就调用
				handler(newVal){
					const findResult = this.options.find(x => x.text === '购物车')
					if(findResult){
						findResult.info = newVal
					}
				},
				immediate:true //页面一加载就调用
			}
		},
		data() {
			return {
				//准备为商品详情页数据
				goods_info:{},
				//配置uni-app自带uni-goods-nav组件
				options:[{
					icon:'shop',
					text:'店铺',
					infoBackgroundColor:'#ff0000',
					infoColor:"white"
				},{
					icon:'cart',
					text:'购物车',
					info:0
				}],
				buttonGroup:[{
					text:'加入购物车',
					backgroundColor:'#ff0000',
					color:'#fff'
				},{
					text:'立即购买',
					backgroundColor:'#ffa200',
					color:'#fff'
				}
			]
			};
		},
		onLoad(options){
			//拿到跳转页面发送过来的参数goods_id
			const goods_id = options.goods_id
			this.getGoodsDetail(goods_id)
		},
		methods:{
			...mapMutations('m_cart',['addToCart']),//映射
			//请求商品详情页面的数据
			async getGoodsDetail(goods_id){
				const {data:res} = await uni.$http.get('/api/public/v1/goods/detail',{goods_id:goods_id})
				if(res.meta.status !== 200) return uni.$showMsg()
				//对请求来的数据进行处理
				res.message.goods_introduce = res.message.goods_introduce.replace(/<img /g,'<img style="display:block;"').replace(/webp/g,'jpg')
				//存入goods_info
				this.goods_info = res.message
			},
			//图片预览功能
			preview(i){
				console.log(this.goods_info.pics)
				uni.previewImage({
					current:i,
					//map返回一个符合条件的新数组
					urls:this.goods_info.pics.map(x => x.pics_big)
				})
			},
			//配置购物车按钮的跳转
			onClick(e){
				if(e.content.text === '购物车'){
					uni.switchTab({
						url:'/pages/cart/cart'
					})
				}
			},
			//配置加入购物车按钮的操作
			buttonClick(e){
				if(e.content.text === '加入购物车'){
					//将当前goods_info里的数据通过mapMutations写入到vuex(m_cart)
					const goods = {
						goods_id:this.goods_info.goods_id,
						goods_name:this.goods_info.goods_name,
						goods_price:this.goods_info.goods_price,
						goods_count:1,
						goods_small_logo:this.goods_info.goods_small_logo,
						goods_state:true
					}
					this.addToCart(goods)
				}
			}
		}
	}
</script>

<style lang="scss">
swiper{
	height:750rpx;
	
	image{
		width: 100%;
		height: 100%;
	}
}
.goods-info-box{
	padding: 10px;
	padding-right: 0;
	
	.price{
		color: #c00000;
		font-size: 18px;
		margin: 10px 0;
	}
	
	.goods-info-body{
		display: flex;
		justify-content: space-between;
		
		.goods-name{
			font-size: 13px;
			margin-right: 10px;
		}
		
		.favi{
			width: 120px;
			font-size: 12px;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			border-left: 1px solid #efefef;
			color: gray;
		}
	}
	.yf{
		font-size: 12px;
		color:gray;
		margin: 10px 0;
	}
}

.goods_nav{
	position: fixed;
	bottom: 0;
	left: 0;
	width: 100%;
	
}
.goods-detail-container{
	padding-bottom: 50px;
}
</style>
