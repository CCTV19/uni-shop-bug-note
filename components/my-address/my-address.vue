<template>
	<view>
		<!--如果从vuex中映射的address为空则显示这个-->
		<view class="address-choose-box" v-if="JSON.stringify(address) === '{}'">
			<button type="primary" size="mini" 
			class="btnChooseAddress"
			@click="chooseAddress">请选择收货地址+</button>
		</view>
		
		<!--如果从vuex中映射的address不为空则显示-->
		<view class="address-info-box" v-else @click="chooseAddress">
			<view class="row1">
				<view class="row1-left">
					<view class="username">收货人：{{address.userName}}</view>
				</view>
				<view class="row1-right">
					<view class="phone">电话：{{address.telNumber}}</view>
					<uni-icons type="arrowright" size="16"></uni-icons>
				</view>
			</view>
			<view class="row2">
				<view class="row2-left">收货地址：</view>
				<view class="row2-right">{{addstr}}</view>
			</view>
		</view>
		<my-settle></my-settle>
	</view>
</template>

<script>
	import {mapState,mapMutations,mapGetters} from 'vuex'
	
	export default {
		name:"my-address",
		data() {
			return {
				/* address:{} */
			};
		},
		methods:{
			//使用辅助函数触发Mutation操作
			...mapMutations('m_user',['updateAddress']),
			async chooseAddress(){
				//调用uni提供的选择地址api
				const [err,succ] = await uni.chooseAddress().catch(err => err)
				if(err === null && succ.errMsg === 'chooseAddress:ok'){
					/* this.address = succ */
					//将存到store中
					this.updateAddress(succ)
				}
			}
		},
		computed:{
			//将当前组件需要的全局数据，映射为当前组件computed属性
			...mapState('m_user',['address']),
			...mapGetters('m_user',['addstr'])
		}
	}
</script>

<style lang="scss">
.address-choose-box{
	height: 90px;
	display: flex;
	justify-content: center;
	align-items: center;
}
.address-info-box{
	font-size: 12px;
	height: 90px;
	display: flex;
	flex-direction: column;
	justify-content: center;
	padding: 0 5px;
	
	.row1{
		display: flex;
		justify-content: space-between;
		
		.row1-right{
			display: flex;
			justify-content: space-between;
		}
	}

	.row2{
		display: flex;
		
		align-items: center;
		margin-top: 10px;
		
		.row2-left{
			white-space: nowrap;
		}
	}
}
</style>