<template>
	<div class="opportunity-wrap">
		<div class="opportunity-topHeaderDiv">
			<img :src="courseLogo" alt="" class="opportunity-img-Grain">
			<div class="opportunity-introduce-Text">{{ courseDetail.courseName }}</div>
			<span class="opportunity-price">拼课价:</span>
			<span class="opportunity-now-money">￥{{ courseDetail.pinPrice }}</span>
			<span class="opportunity-oldPrice">原价:￥{{ courseDetail.coursePrice }}</span>
			<img src="../../assets/6.png" height="25" width="65" class="opportunity-flag">
			<div class="opportunity-peopleNumberBox">{{ courseDetail.pinNum }}人团</div>
		</div>


		<!-- 分隔块1 -->
		<div class="opportunity-interval"></div>

		<!-- 拼课名额部分 -->
		<div class="opportunity-places">
			<img src="../../assets/7.png" height="16" width="15" class="opportunity-hourglass">
			<span class="opportunity-placesText" v-show="!courseDetail.surplus">拼课机会已被抢光</span>
			<span class="opportunity-placesText" v-show="courseDetail.surplus">还有{{ courseDetail.surplus }}个拼课机会</span>
			<button class="opportunity-success" v-show="courseDetail.surplus" @click="pay">我要拼课</button>
		</div>

		
		


		<!-- 分隔块2 -->
		<div class="opportunity-interval-T"></div>


		<!-- 拼课说明部分 -->
		<div class="opportunity-places">
			<img src="../../assets/8.png" height="16" width="16" class="opportunity-hourglass">
			<span class="opportunity-placesText">拼课说明</span>
		</div>

		<!-- 拼课说明介绍部分 -->
		<div class="opportunity-introduceBox">
			<p>拼团是一种快速有效的营销活动，主要玩法是要在限定时间内，通过分享产品链接，找到规定的人数一起购买才能完成拼团否则失败，举个例子，一件商品拼团时间24小时，5人拼团，你是团长，付款后要通过分享的链接在24小时之内找到另外4个购买者才行！拼团的商品价格要比原价低。</p>
		</div>


		<!-- 分隔块3 -->


		<!-- 进入听课按钮部分 -->
		
	</div>
</template>
<script type="text/javascript">
import { mapState } from 'vuex';
export default {
	computed: {
        ...mapState({
            common_request_base_url: state => state.common.common_request_base_url,
            userinfo_data : state => state.UserInfo.userinfo_data,
            common_request_appendv1_url: state => state.common.common_request_appendv1_url,
        })
    },
	data() {
		return {
			courseDetail: {
				type: Object
			},
			opportunity: {},
			isbuy: false,
			courseLogo: '',
			userId: ''
		};
	},
	methods: {
		pay() {
			// 拼课支付
			var params = {
				id: localStorage.getItem('dataid'),
				ubrPayType: '0',
				// 课程type是2，拼课
				ubrCourseType: '2',
				ubrBuyNum: '1',
				ubrUserId: this.userId,
				ubrCoursePrice: this.courseDetail.pinPrice,
				ubrCourseId: this.$route.params.opportunity.courseId,
				openid: localStorage.getItem('openid')
			};
			console.log(params);
			let url = this.common_request_base_url + 'v1/app/createorder';
			var qs = require('qs');
			this.$http.post(url, qs.stringify(params),{
				 headers: {
    				'Content-Type': 'application/x-www-form-urlencoded',
  				}
			}).then((res) => {
				console.log(res);
				let money = this.courseDetail.pinPrice * 100;
				let payurl = this.common_request_base_url + 'wx/wxpay?id=' + localStorage.getItem('dataid') + '&orderNumber=' + res.data.content.result.orderNum + '&total_fee=' + money + '&openid=' + localStorage.getItem('openid') + '&goods_body=' + this.courseDetail.courseName + '&goods_attach=' + this.courseDetail.courseName + '['+ this.$store.state.UserInfo.userId + ']' + '&goods_tag=' + '青枝';
					this.$http.get(payurl)
				.then((res) => {
					let payload = {};
					payload.timestamp = res.data.timeStamp;
					payload.appId = res.data.appId;
					payload.nonceStr = res.data.nonceStr;
					payload.package = res.data.package;
					payload.paySign = res.data.paySign;
					payload.signType = res.data.signType;
					payload.success_callback = this.paySuccess;
					this.$store.dispatch('wechatPay', payload);
				})
				.catch((error) => {
					this.$toast('服务端异常');
				});
			});
		},
		paySuccess(res) {
			this.$toast('拼课成功');
		}
	},
	mounted() {
		console.log(this.$route.params);
	},
	props: {
		userId: ''
	},
	beforeRouteEnter (to, from, next) {
		next((vm) => {
			console.log(vm.$route.params);
			console.log(vm.$router);
			console.log(vm.$store.state);
			console.log(vm.$route.params.userId);
			vm.userId = vm.$route.params.userId;
			let id = localStorage.getItem('dataid');
			let url = vm.common_request_base_url + vm.common_request_appendv1_url + 'findgroupcoursebycourseid?id=' + id + '&courseId=' + vm.$route.params.opportunity.courseId + '&openid=' + localStorage.getItem('openid');
			vm.$http.get(url)
			.then((res) => {
				console.log(res);
				vm.courseDetail = res.data.content.result;
				let url = vm.common_request_base_url + vm.common_request_appendv1_url + 'getossmedia?media=' + vm.courseDetail.courseLogo + '&openid=' + localStorage.getItem('openid');
				vm.$http.get(url)
				.then((res) => {
					vm.courseLogo = res.data
				})
				.catch((error) => {
					vm.$toast('获取拼课课程头像失败');
				});
			});
		});
	}
};
</script>
<style type="text/css" scroped>
	body{
		background-color:#fafafa;
	}
	.opportunity-wrap{
		width:100%;
		height:100%;
		max-width:750px;
	}
	.opportunity-wrap .opportunity-topHeaderDiv{
		width:100%;
		height:8rem;
		margin:0 auto;
		border-top:0.01rem solid #eee;
		position:relative;
		background-color:white;
	}
	.opportunity-wrap .opportunity-topHeaderDiv .opportunity-img-Grain{
		width:6rem;
		height:6rem;
		margin-top:0.94rem;
		margin-left:0.625rem;
		border-radius:2px;
	}
	.opportunity-introduce-Text{
		width:66.2%;
		position:absolute;
		top:1.8rem;
		left:7rem;
		font-size:0.89rem;
		word-break:break-all;
		font-family:"Microsoft Yahei";
		color:#333333;
		line-height:1.2rem;
		overflow: hidden;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-line-clamp: 2;
		-webkit-box-orient: vertical;
		/*font-weight:bold;*/
	}
	.opportunity-price{
		color:#ff2626;
		font-size:0.625rem;
		position:absolute;
		top:5.4rem;
		left:7rem;
	}
	.opportunity-now-money{
		color:#ff2626;
		font-size:1.125rem;
		font-family:"Microsoft YaHei";
		position:absolute;
		left:9.3rem;
		top:5.1rem;
	}
	.opportunity-oldPrice{
		text-decoration:line-through;
		color:#999999;
		position:absolute;
		top:5.3rem;
		left:14rem;
		font-size:0.625rem;
	}
	.opportunity-peopleNumberBox{
		width:3.4rem;
		height:1.4rem;
		position:absolute;
		top:5.3rem;
		right:0;
		font-size:0.7rem;
		color:white;
		text-align:center;
		line-height:1.4rem;
		font-family:"Microsoft YaHei";
	}

	.opportunity-flag{
		position:absolute;
		right:0;
		top:5rem;
	}
	@media screen and (max-width: 375px) {
			.opportunity-introduce-Text{
				width:66.2%;
				position:absolute;
				left:7rem;
				
			}

	}
	@media screen and (max-width: 320px) {
		.opportunity-wrap .opportunity-topHeaderDiv .opportunity-img-Grain{
			width:5.6rem;
			height:6rem;
			margin-top:0.9rem;
			margin-left:0.5rem;
		}
		.opportunity-price {
		    color: #ff2626;
		    font-size: 0.2rem;
		    position: absolute;
		    top: 5.4rem;
		    left: 6.6rem;
		}
		.opportunity-introduce-Text{
			width:66.2%;
			position:absolute;
			left:6.6rem;
			

		}
		.opportunity-now-money{
			font-size:1.125rem;
			color:#ff2626;
			font-family:"Microsoft Yahei";
			position:absolute;
			left:140px !important;
			top:74px !important;
		}
		.opportunity-oldPrice{
			text-decoration:line-through;
			color:#999999;
			position:absolute;
			top:5.3rem;
			left:12.6rem;
			font-size:0.625rem;
		}
		.opportunity-WXNameText {
		    font-size: 1rem;
		    position: absolute;
		    top: 1rem;
		    left: 4.8rem;
		}
	}



	/*分隔块1部分*/
	.opportunity-interval{
		width:100%;
		height:0.625rem;
		border-top:1px solid #e5e5e5;
		border-bottom:1px solid #e5e5e5;
	}



	/*评课名额部分*/
	.opportunity-places{
		width:100%;
		height:3rem;
		border-bottom:1px solid #e7e7e7;
		position:relative;
		background-color:white;
	}
	.opportunity-hourglass{
		margin-top:1rem;
		margin-left:0.625rem;
	}

	.opportunity-placesText{
		font-size:0.94rem;
		position:absolute;
		line-height:3rem;
		left:2rem;
		font-family:"Microsoft Yahei";
		color:#333333;
	}
	.opportunity-success{
		background-color:#ff2626;
		width:65px;
		height:26px;
		border:none;
		border-radius:5px;
		position:absolute;
		top:0.69rem;
		right:1.25rem;
		line-height:26px;
		font-size:0.81rem;
		color:white;
	}

	/*用户信息部分*/
	

	/*分隔块2部分*/
	.opportunity-interval-T{
		width:100%;
		height:0.625rem;
		border-bottom:1px solid #e5e5e5;
	}

	/*文字介绍部分*/
	.opportunity-introduceBox{
		width:100%;
		border-bottom:1px solid #e5e5e5;
		padding-top:0.4rem;
		padding-bottom:0.6rem;
		background-color:white;
	}
	.opportunity-introduceBox p{
		width:94%;
		margin:0 auto;
		line-height:1.5rem;
		font-size:0.75rem;
		color:#999999;
		font-family:"Microsoft Yahei";
	}

	


	/*进入听课部分*/
	.opportunity-into{
		width:100%;
		height:3.5rem;
		background-color:red;
		text-align:center;
		line-height:3.5rem;
		color:white;
		font-size:1.6rem;
		position:fixed;
		bottom:0;
	}
</style>
