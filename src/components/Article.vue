<template>
	<article>
		<div v-if="!articleData.content" class="box-bj-sd warp">
			<span>暂无内容</span>
		</div>
		<div v-else class="box-bj-sd warp">
			<h1>{{articleData.title}}</h1>
			<ul class="meta">
				<li>
					<i class="fa fa-calendar"></i>
					发布时间：{{articleData.publishTime}}
				</li>
				<!-- <li>
					<i class="fa fa-user fa-lg"></i>
					发布者：{{articleData.author}}
				</li> -->
				<li class="viewNum">
					<i class="fa fa-eye fa-lg"></i>
					围观数：{{articleData.viweNum}}
				</li>
				<li>
					<i class="fa fa-thumbs-o-up fa-lg"></i>
					点赞：{{starNum}}
				</li>
			</ul>
			<div class="article-meta">
				<div class="inner" v-html="articleData.content"></div>
				<p style="text-align: center;margin-top: 25px;">
					<font>- - - END - - -</font>
				</p>
				<p style="text-align: center;margin-top: 15px;">
					<font>
						浏览完了？你可以
						<a href="#word" style="text-decoration: none;color: #096">点我去评论</a>留下观点！
					</font>
				</p>
				<!-- tag标签 -->
				<p class="tags">
					<i data-v-0994aea9 class="fa fa-tags"></i>：
					<!-- <a
            href="javascript:void(0);"
            v-for="(tag,index) in articleData.keywords"
            :key="index"
            :style="'background-color:'+ getRandomColor()+';opacity: 0.8;'"
					>{{tag}}</a>-->
					<router-link
						v-for="(tag,index) in articleData.keywords"
						:key="index+'sd'"
						to="/tags"
						@click.native="handleTo(tag)"
						:style="'background-color:'+ getRandomColor()+';opacity: 0.8;'"
					>{{tag}}</router-link>
				</p>
				<p class="share-url">
					<!-- <b>转载：</b>
          本站原创文章由LING☆璐个人发表，如需转载敬请将本文链接作为出处标注，谢谢合作！
					<br />-->
					<span>本文链接地址：</span>http://zhenglinglu.cn/detail/?id={{bid}}
					<!-- <a :href="'https://zhenglinglu.cn/article?bid='+bid" target="_blank">{{articleData.title}}</a> -->
				</p>
				<div class="post-like-donate">
					<!-- <span class="diggit" @click.stop="handleClickStar">
            <a href="JavaScript:void(0);">棒棒哦！</a>(
            <b id="diggnum">{{starNum}}</b> )
					</span>-->
					<!-- <a href="javascript:void(0);" class="donate">
						<i class="fa fa-bitcoin"></i> 打赏
					</a> -->
					<a href="javascript:void(0);" @click.stop="handleClickStar" class="Love">
						<i class="fa fa-thumbs-o-up"></i> 点赞(<span class="love-count"> {{starNum}} </span>)
					</a>
					<!-- <a href="javascript:void(0);" class="share">
						<i class="fa fa-share-alt"></i> 分享
					</a> -->
				</div>
			</div>
		</div>
		<div class="page box-bj-sd">
			<p>
				上一篇：
				<a
					v-if="prevArticle"
					@click.stop="handleLook(prevArticle.bid)"
					href="javascript:void(0);"
				>{{prevArticle.title}}</a>
				<a v-else href="javascript:void(0);">没有了！</a>
			</p>
			<p>
				下一篇：
				<a
					v-if="nextArticle"
					@click.stop="handleLook(nextArticle.bid)"
					href="javascript:void(0);"
				>{{nextArticle.title}}</a>
				<a v-else href="javascript:void(0);">没有了！</a>
			</p>
		</div>
		<div class="guess box-bj-sd">
			<h3>猜你喜欢以下内容</h3>
			<ul>
				<li v-for="item in guessArticle" :key="item.bid" @click.stop="handleLook(item.bid)">
					<img :src="$url+'/'+item.imgSrc" :alt="item.title" />
					<span>{{item.title}}</span>
				</li>
			</ul>
		</div>
		<div class="say-box" id="word">
			<ArticleWord :word="wordObj" />
		</div>
		<transition name="carouselBgc">
			<div class="bgc" v-if="isCarouselBgc"></div>
		</transition>
	</article>
</template>
<script>
	import ArticleWord from "./ArticleWord";
	// import hljs from "highlight.js";
	// import Vue from "vue";
	// import { URL } from "../constant/constant.js";
	// import { goBack } from "../assets/gloable.js";
	// import Qs from "qs";
	export default {
		name: "article-sing",
		components: {
			ArticleWord
		},
		head() {
			return {
				title: this.articleData ? this.articleData.title : "",
				meta: [
					{
						hid: "description",
						name: "description",
						content: this.articleData ? this.articleData.description : ""
					},
					{
						hid: "keywords",
						name: "keywords",
						content: this.articleData ? this.articleData.keywords : ""
					}
				]
			};
		},
		data: () => ({
			articleData: {},
			prevArticle: {},
			nextArticle: {},
			guessArticle: {},
			bid: "",
			wordObj: {},

			title: "",
			isStatic: false,
			isClickStar: false,
			isCarouselBgc: false,
			// URL: this.$url + "/detail/?id=",
			// -------------
			data: {},

			index: 0,

			starNum: 0,
			count: 1 //点赞计数
		}),
		// props: {
		//   bid: {
		//     type: String,
		//     default: ""
		//   },
		//   article: {
		//     type: Object,
		//     default: {}
		//   },
		//   static: {
		//     type: Boolean,
		//     default: false
		//   }
		// },
		computed: {
			articleId() {
				return this.$route.query.id;
			}
			//   articleAll() {
			//     return this.$store.state.article.articleAll;
			//   }
			//   // singleArticle() {
			//   //   return this.$store.state.article.singleArticle;
			//   // }
		},
		watch: {
			articleId(data) {
				this.handleData();
			}
			//   article(data) {
			//     this.handleData(this.bid, data);
			//   },
			//   bid(bid) {
			//     //监听文章数据
			//     this.handleData(bid);
			//   },
			//   articleAll() {
			//     //监听文章数据
			//     this.handleData(this.bid);
			//   }
		},
		created() {
			// this.isStatic = this.static;

			this.handleData();

			// this.$store.dispatch("setSingleArtile", this.$route.query.bid);
			// //首页通知
			// this.$store.dispatch("setChangingOver", { notice: false, position: true });
			// // 侧边栏初始化
			// this.$store.dispatch("setAsideSingleConfigsData", "b");
			// this.data = this.$store.state.article.singleArticle;
			// this.starNum = this.data.starNum;
			// this.bid = this.data.bid;
			// this.title = this.data.title;
			// this.getPrevNext();
		},
		methods: {
			handleData() {
				this.$event.emit("inLoading", true);
				this.bid = this.$route.query.id;
				this.$axios
					.get(this.$url + "/zll/article/list", {
						params: { id: this.bid }
					})
					.then(res => {
						if (res.data.result) {
							let articleData = res.data.data.article;
							if (articleData.content.indexOf('src="images/') !== -1) {
								articleData.content = articleData.content.replace(
									/src="images\//g,
									'src="' + this.$url + "/" + "images/"
								);
							}
							articleData.publishTime = articleData.publishTime.slice(0, 10);
							this.articleData = articleData;
							this.starNum = articleData.starNum;
							this.prevArticle = res.data.data.prev;
							this.nextArticle = res.data.data.next;
							this.guessArticle = res.data.data.guess;
							this.wordObj = {
								id: this.bid,
								title: this.articleData.title
							};
						}
						this.$event.emit("inLoading", false);
						let time = window.setTimeout(() => {
							window.clearTimeout(time);
							this.$event.emit("pageLoading", false);
						}, this.$loadingTime);
					})
					.catch(error => {
						console.log(error);
					});
				// if (this.isStatic) {
				//   article.middle.keywords = article.middle.keywords.replace("、", "，");
				//   article.middle.keywordsArr = article.middle.keywords.split("，");
				//   this.prevArticle = article.prev;
				//   this.articleData = article.middle;
				//   this.nextArticle = article.next;
				// } else {
				//   this.id = bid;
				//   var data = this.$store.state.article.articleAll;
				//   let length = data.length;
				//   if (length !== 0 && bid) {
				//     for (let index = 0; index < length; index++) {
				//       if (data[index].bid === bid) {
				//         this.prevArticle = index !== 0 ? data[index - 1] : false;
				//         this.articleData = data[index];
				//         //  console.log(this.articleData);
				//         // this.articleData.keywords = this.articleData.keywords.replace("、","，");
				//         this.articleData.keywordsArr = this.articleData.keywords;
				//         this.nextArticle = index !== length - 1 ? data[index + 1] : false;
				//         break;
				//       }
				//     }
				//   }
				// }
			},
			// 标签跳转
			handleLook(bid) {
				// 将bid存储到store中
				// this.$store.commit("setBid", bid);
				// if (!isStatic) {
				// this.$router.push({ path: "/detail/" + bid });
				// }
				// this.$store.dispatch("setSingleArtile", bid);

				this.$router.push({ path: "/detail", query: { id: bid } });
				// 返回顶部
				this.$goBack();
			},
			//跳转到详情页
			handleTo(name) {
				this.$store.commit("setTagValue", name);
				// 返回顶部
				this.$goBack();
				// this.$store.commit("setSearchValue", { name, url });
			},

			//点赞
			handleClickStar() {
				 this.$Message.destroy();
				if (!this.isClickStar) {
					let data = {
						id: this.bid
					};
					this.$axios
						.post(this.$url + "/zll/article/update/star",this.$qs.stringify(data))
						.then(res => {
							this.starNum = this.starNum * 1 + 1;
							this.$Message["success"]({
								background: true,
								content: "谢谢您的支持！٩(๑>◡<๑)۶ "
							});
							this.isClickStar = true;
							// console.log(res);
						})
						.catch(error => {
							console.log(error);
						});
				} else {
					this.$Message["info"]({
						background: true,
						content: "您已经点过赞啦！٩(๑>◡<๑)۶ "
					});
				}
			},

			// 获取随机颜色
			getRandomColor() {
				return (
					"#" +
					("00000" + ((Math.random() * 16777215 + 0.5) >> 0).toString(16)).slice(
						-6
					)
				);
			}
		}
	};
</script>
<style lang="scss" scoped>
	article {
		word-wrap: break-word;
		color: #657487;
		line-height: 1.9;
		font-size: 15px;
		.warp {
			padding: 12px;
			margin-bottom: 12px;
		}
		border-radius: 3px;
		margin-bottom: 20px;
		h1 {
			margin-top: 20px;
			color: #666;
			text-align: center;
			font-size: 25px;
			line-height: 33px;
			font-weight: bold;
			margin-bottom: 12px;
			padding: 12px 0;
		}

		ul.meta {
			position: relative;
			margin-top: 25px;
			// background-color: #f8f8f8;
			border-top: 1px dashed #999;
			border-bottom: 1px dashed #999;
			font-size: 14px;
			padding: 5px 0px;
			display: flex;
			// margin-bottom: 15px;
			margin-right: 10px;
			margin-left: 10px;
			justify-content: center;
			li {
				padding: 0 20px;
				margin: 0px 5px;
				font-size: 13px;
				// color: #606060;
				i {
					margin-right: 5px;
				}
			}
			// li:nth-child(1) i {
			//   color: #009966;
			// }
			// li:nth-child(2) i {
			//   color: #f8b06b;
			// }
			// li:nth-child(3) i {
			//   color: #add895;
			// }
			// li:nth-child(4) i {
			//   color: #e4aeae;
			// }
		}
		.article-meta {
			padding-top: 25px;
			// border-top: 1px solid #eee;
			// .code-highlight {
			//   padding-bottom: 70px;
			// }
			p.tags {
				padding-left: 20px;
				margin-top: 15px;
				font-weight: 600;
				a {
					padding: 5px 10px;
					margin-right: 10px;
					border-radius: 3px;
					color: #fff;
					font-size: 13px;
				}
				a:hover {
					opacity: 1 !important;
					transform: scale(1.1);
				}
			}

			.explain {
				width: 95%;
				border-radius: 3px;
				line-height: 28px;
				padding: 15px 10px;
				background-color: #fffce7;
				border: 1px dashed #9acbe8;
				color: #ff4000;
				margin: 0 8px;
				a {
					color: #666;
				}
			}
			.share-url {
				margin: 25px 20px;
				padding: 10px 0px;
				background: #f7f7f7;
				border-left: 3px solid #000;
				span {
					margin-left: 25px;
				}
			}
		}

		.say-box {
			padding-top: 20px;
			// border-top: 1px dashed #aaa;
			p {
				text-align: right;
				margin-right: 30px;
			}
		}
		.post-like-donate {
			display: flex;
			justify-content: center;
			padding-bottom: 10px;
			a {
				width: 130px;
				height: 40px;
				font-size: 14.5px;
				line-height: 40px;
				text-align: center;
				transition: all 0.3s ease;
				text-decoration: none;
				margin: 0px 10px;
			}
			a.Love {
				border: 1px solid #e87461;
				color: #e87461;
			}
			a.Love:hover {
				background: #e87461;
				color: #fff;
			}
			a.donate {
				border: 1px solid #3496e6;
				color: #3496e6;
			}
			a.donate:hover {
				background: #3496e6;
				color: #fff;
			}
			a.share {
				border: 1px solid #78ce79;
				color: #78ce79;
			}
			a.share:hover {
				background: #78ce79;
				color: #fff;
			}
		}
		.guess {
			margin-top: 20px;
			padding: 20px;
			h3 {
				padding-bottom: 10px;
				font-weight: bold;
				font-size: 16px;
				border-bottom: 1px solid #ccc;
			}
			ul {
				display: flex;
				flex-wrap: wrap;
				margin-top: 10px;
				li {
					width: 31%;
					margin: 7px;
					position: relative;
					cursor: pointer;
					img {
						width: 100%;
						padding: 2px;
						border: 1px solid #e7e7e7;
						height: 182px;
					}
					span {
						position: absolute;
						padding: 2px 10px;
						bottom: 11px;
						left: 1px;
						width: 99%;
						// color: #fff;
						background: rgba(217, 217, 217, 0.6);
						overflow: hidden;
						text-overflow: ellipsis;
						white-space: nowrap;
					}
				}
			}
		}
		.page {
			// border-top: 1px dashed #aaa;
			padding: 15px 60px;
			display: flex;
			justify-content: space-between;
			p {
				a {
					color: #666;
				}
				a:hover {
					color: #2196f3;
				}
			}
		}
	}
</style>
