<template>
  <div class="word">
    <!-- 输入框开始 -->
    <div class="comments-box" v-if="!isShowModal">
      <div class="comments-img">
        <img :src="$url+'/'+imgUrl" />
      </div>
      <div class="inner" @click="makeFaceClose">
        <div class="input-box">
          <label>
            <Input
              prefix="ios-contact"
              v-model="name"
              placeholder="请输入您的称呼..."
              class="input-width"
            />
            <i>*</i>
          </label>
          <!-- <label>
            <Input prefix="ios-mail" v-model="email" placeholder="请输入您的邮箱..." class="input-width" />
          </label>-->
          <label>
            <Input prefix="md-at" v-model="webUrl" placeholder="请输入您的网址..." class="input-width" />
          </label>
        </div>
        <div class="comments-warp">
          <!-- 发布内容输入框，利用Html5的新属性contenteditable，实现可编辑文本 ，会自动将插入的IMG标签解析-->
          <div class="publish_container">
            <p contenteditable="true" id="input_conta"></p>
          </div>
          <div class="comments-textarea">
            <!-- 表情容器 ，包裹生成的表情，绑定点击表情事件-->
            <div class="face-warp">
              <i @click.stop="makeFace" :class="isFaceShow?'i1':'i2'"></i>
              <transition name="show-face">
                <div
                  id="face"
                  class="box-bj-sd"
                  v-show="isFaceShow"
                  @click.stop="choiceFace($event)"
                >
                  <img v-for="(itemIcon,ind) in arrIcon" :src="itemIcon.name" :key="ind+'icons'" />
                </div>
              </transition>
            </div>
            <span class="submit" @click.stop="handlePublic">发布</span>
          </div>
        </div>
      </div>
    </div>
    <div class="massage">
      <span>评论内容</span>
      <p>
        <span>{{total}}</span>条评论
      </p>
    </div>
    <div v-if="replyData.length">
      <div class="leave-list">
        <ul>
          <li v-for="item in replyData" :key="item.bid">
            <div class="list" :class="item.uid?'is-reply':''">
              <!-- <img :src="URL+item.url" alt="头像" /> -->
              <img :src="$url+'/'+item.url" />
              <div class="text-box">
                <div class="user-name">
                  <h3 v-if="!item.uid">
                    {{item.name}}
                    <span># 第{{item.id}}楼</span>
                  </h3>
                  <h3 v-else class="reply-title">
                    {{item.name}}
                    <span>回复</span>
                    {{item.reply}}
                  </h3>
                  <div>
                    <span>发布于：</span>
                    {{item.time.slice(0,10)}}
                  </div>
                </div>
                <div class="text" v-html="item.content"></div>
                <div class="repy">
                  <span @click.stop="handleReply(item.uid?item.uid:item.bid,item.name)">
                    <i class="fa fa-mail-forward"></i>
                  </span>
                  <span @click.stop="handletHumbs(item.bid,item.uid?1:0)">
                    <i class="fa fa-thumbs-o-up" style="margin-right:3px;"></i>
                    {{item.starNum}}
                  </span>
                </div>
              </div>
            </div>
          </li>
        </ul>
      </div>
      <div class="page">
        <Page
          show-total
          @on-change="changePage"
          @on-page-size-change="changeSizePage"
          :total="total"
          show-sizer
        />
      </div>
    </div>
    <div v-else class="leave-list tip">暂无评论内容</div>

    <!-- 回复评论输入内容 -->
    <Modal v-model="isShowModal" width="60%" footer-hide>
      <p slot="header" style="color:#f60;text-align:center">
        <span>回复 @{{replyName}}</span>
      </p>
      <!-- 输入框开始 -->
      <div class="comments-box" style="margin-bottom:30px;">
        <div class="comments-img">
          <img :src="$url+'/'+imgUrl" />
        </div>
        <div class="inner" @click="makeFaceClose">
          <div class="input-box">
            <label>
              <Input
                prefix="ios-contact"
                v-model="name"
                placeholder="请输入您的称呼..."
                class="input-width"
              />
              <i>*</i>
            </label>
            <!-- <label>
              <Input
                prefix="ios-mail"
                v-model="email"
                placeholder="请输入您的邮箱..."
                class="input-width"
              />
            </label>-->
            <label>
              <Input prefix="md-at" v-model="webUrl" placeholder="请输入您的网址..." class="input-width" />
            </label>
          </div>
          <div class="comments-warp">
            <!-- 发布内容输入框，利用Html5的新属性contenteditable，实现可编辑文本 ，会自动将插入的IMG标签解析-->
            <div class="publish_container">
              <p contenteditable="true" id="input_conta"></p>
            </div>
            <div class="comments-textarea">
              <!-- 表情容器 ，包裹生成的表情，绑定点击表情事件-->
              <div class="face-warp">
                <i @click.stop="makeFace" :class="isFaceShow?'i1':'i2'"></i>
                <transition name="show-face">
                  <div
                    id="face"
                    class="box-bj-sd"
                    v-show="isFaceShow"
                    @click.stop="choiceFace($event)"
                  >
                    <img v-for="(itemIcon,ind) in arrIcon" :src="itemIcon.name" :key="ind+'icons'" />
                  </div>
                </transition>
              </div>
              <span class="submit" @click.stop="handlePublic">发布</span>
            </div>
          </div>
        </div>
      </div>
    </Modal>
  </div>
</template>
<script>
// import { URL } from "../constant/constant.js";
import {
  dateTime,
  // getUrl,
  checkEmail,
  getuid,
  analysisString,
  icon
} from "../assets/js/globle";
// import Qs from "qs";
export default {
  name: "wordsa",
  data: () => ({
    sayList: [], //部分数据
    content: [], //全部数据
    pageNo: 1,
    pageSize: 10,
    total: 0,
    bid: "",
    replyName: "",
    // repybid: "", //回复id 用于判断
    uid: "", ///回复id传向后台修改
    // repyNameHou: "", //回复谁
    arrIcon: [],
    colorArr: [],
    URL: process.env.baseUrl,
    isClickStar: [],
    //----------------
    isFaceShow: false,
    isShowModal: false,

    repyisFaceShow: false,
    // content: "",
    replyData: [],
    // total: 0,
    email: "",
    name: "",
    webUrl: "",
    repyemail: "",
    repyname: "",
    repywebUrl: "",

    imgUrl: "",
    //--------------------

    time: dateTime(),
    mark: true
  }),
  props: ["config"],
  // computed: {
  //   commentAll() {
  //     return this.$store.state.comment.commentAll;
  //   }
  // },
  // watch: {
  //   commentAll() {
  //     this.handleData();
  //   }
  // },

  created() {
    // this.bid = getuid();
    // console.log(this.bid);
    // this.artBid = this.$route.params.bid;
    // this.handleData();

    // this.imgUrl = this.$url + "/images/headimg/mo.jpg";

    this.getContent();
  },
  mounted() {
    this.imgUrl = this.randomImg(1, 25);

    this.handleIcon();
  },
  methods: {
    handleIcon() {
      this.arrIcon = icon.map(item => {
        // item.name = "https://zhenglinglu.cn/staticimg/icon/" + item.name;
        return {
          name: this.$url + "/images/emoji/" + item.name
        };
      });
    },
    randomImg(m, n) {
      let random = Math.floor(Math.random() * (m - n) + n);
      return "images/headimg/head" + random + ".jpg";
    },
    // handleData() {
    //   let data = this.$store.state.comment.commentAll;
    //   // console.log(data);
    //   this.content = data;
    //   this.total = data.length;

    //   this.sayList = data.slice(0, 10);
    // },
    changePage(event) {
      this.pageNo = event;
      this.getContent();
      // this.changeList(this.pageNo, this.pageSize);
      // 返回顶部
      this.$goBack();
    },
    changeSizePage(event) {
      this.pageSize = event;
      this.getContent();
      // this.changeList(this.pageNo, this.pageSize);
      // 返回顶部
      this.$goBack();
    },
    // changeList(pageNo, pageSize) {
    //   let start = pageNo,
    //     end = pageSize;
    //   if (pageNo > 0) {
    //     start = (pageNo - 1) * pageSize;
    //     end = start + pageSize;
    //   }
    //   this.sayList = this.content.slice(start, end);
    // },
    makeFaceClose() {
      if (this.repybid) {
        this.repyisFaceShow = false;
      } else {
        this.isFaceShow = false;
      }
    },
    makeFace() {
      if (this.repybid) {
        this.repyisFaceShow = !this.repyisFaceShow; //显示表情容器
      } else {
        this.isFaceShow = !this.isFaceShow; //显示表情容器
      }
      if ($("#face>img").length == 0) {
        //动态生成表情，如果现在没有表情则生成
        for (var i = 1; i <= 75; i++) {
          //根据表情文件数量决定循环次数，这里为75个表情
          $("#face").append('<img src="/static/arclist/' + i + '.gif">'); //为表情容器里添加IMG标签，并赋予src值，路径为表情文件所在路径
        }
      }
    }, // 选择表情并插入到输入框
    choiceFace(e) {
      if (e.target.nodeName == "IMG") {
        let choice = e.target;
        let cEle = choice.cloneNode(true); //深度复制，复制节点下面所有的子节点 ，直接将整个表情的IMG标签复制，并添加到发布框的<p></p>里面
        $("p#input_conta").append(cEle);
        // let cEle = e.target.cloneNode(true);
        // let src = cEle.src;
        // let start = src.indexOf("images"),end = src.indexOf(".gif");
        // let img = `<img src="${src.slice(start, end)}.gif">`;
        // $("p#input_conta" + this.uid).append(img);
        this.makeFaceClose();
      }
    },
    // 请求留言数据
    getContent() {
      this.$Message.destroy();
      let data = {
        pageNo: this.pageNo,
        pageSize: this.pageSize
      };
      this.$axios
        .get(this.$url + "/zll/blog/word", { params: data })
        .then(res => {
          if (res.data.result) {
            let data = res.data.list;
            data.forEach(item => {
              let content = item.content;
              if (content.indexOf("<img src=") !== -1) {
                content = content.replace(
                  /<img src="/g,
                  '<img src="' + this.$url + "/"
                );
              }
              item.content = content;
            });
            this.replyData = data;
            this.total = res.data.count;
            // let data = res.data.list;
            // $("#input_conta" + this.uid).html(""); //清除发布框的文本内容
            // this.name = "";
            // this.email = "";
            // this.webUrl = "";
            // this.$store.commit("setCommentAll", data);
            // this.handleData();
            // this.bid = getuid();
          } else {
            this.$Message["error"]({
              background: true,
              content: "数据加载失败！呜呜~"
            });
          }
          let time = window.setTimeout(() => {
            window.clearTimeout(time);
            this.$event.emit("pageLoading", false);
          }, this.$loadingTime);
        });
    },
    //回复按钮
    // handleRepy(bid, name, uid) {
    //   // this.repyNameHou = name;
    //   // this.bid = getuid();
    //   // if (this.repybid) {
    //   //   this.uid = "";
    //   //   this.repybid = "";
    //   // } else {
    //   //   this.repybid = bid;
    //   //   this.uid = uid; //用于匹配回复内容
    //   // }
    // },
    //提交数据
    handlePublic() {
      this.$Message.destroy();
      $("#input_conta img").each(function() {
        let src = $(this).attr("src");
        let start = src.indexOf("images"),
          end = src.indexOf(".gif");
        let img = `<img src="${src.slice(start, end)}.gif">`;
        $(this).replaceWith(img);
      });
      let text = $("#input_conta" + this.uid).html(); //获得发布框的文本内容，表情会以整个img标签文本显示

      let time = dateTime();
      if (!this.name) {
        this.$Modal.info({
          title: "温馨提示",
          content: "请输入您的称呼！"
        });
      }
      // else if (!this.email) {
      //   this.$Modal.info({
      //     title: "温馨提示",
      //     content: "请输入您的邮箱地址！"
      //   });
      // }
      // else if (!checkEmail(this.email)) {
      //   this.$Modal.info({
      //     title: "温馨提示",
      //     content: "邮箱格式不正确！"
      //   });
      // }
      else if (!text) {
        this.$Modal.info({
          title: "温馨提示",
          content: "请输入评论内容！"
        });
      } else {
        if (this.mark) {
          this.$Message.loading({
            content: "数据正在提交...",
            duration: 0
          });
          let data = {};
          this.mark = false;
          data.mark = 1;
          data.name = this.name;
          data.content = text;
          data.email = this.email;
          data.url = this.imgUrl;
          data.web = this.webUrl;
          data.id = this.total ? this.total * 1 + 1 : 0;
          if (this.replyName) {
            data.reply = this.replyName;
          }

          let url = "/zll/blog/word";
          if (this.bid) {
            url = "/zll/blog/word/reply";
            data.uid = this.bid;
          }

          this.$axios
            .post(this.$url + url, this.$qs.stringify(data))
            .then(res => {
              this.$Message.destroy();
              if (res.data.result) {
                this.name = "";
                this.email = "";
                this.imgUrl = this.randomImg(1, 25);
                this.webUrl = "";
                this.$nextTick(() => {
                  $("#input_conta").html(""); //清除发布框的文本内容
                });
                this.getContent();
                this.$Message["success"]({
                  background: true,
                  content: "留言提交成功！٩(๑>◡<๑)۶ "
                });
                this.bid = "";
                this.replyName = "";
                this.isShowModal = false;
                this.mark = true;
              } else {
                this.$Message["error"]({
                  background: true,
                  content: "留言提交失败！o(╥﹏╥)o "
                });
                this.mark = true;
              }
            })
            .catch(error => {
              console.log(error);
            });
        } else {
          this.$Message.error("数据正在提交...");
        }
      }
    },

    // 更新点赞留言
    handletHumbs(id, mark) {
      this.$Message.destroy();
      let arrStar = this.isClickStar,
        state = false;
      for (let index = 0; index < arrStar.length; index++) {
        if (arrStar[index] === id) {
          state = true;
          break;
        }
      }
      if (state) {
        this.$Message["info"]({
          background: true,
          content: "您已经点过赞啦！٩(๑>◡<๑)۶ "
        });
      } else {
        this.isClickStar.push(id);
        let data = { id };
        if (mark === 1) {
          data.reply = true;
        }
        this.replyData = this.replyData.map(item => {
          if (item.bid === id) {
            item.starNum = item.starNum * 1 + 1;
          }
          return item;
        });
        this.$axios
          .post(this.$url + "/zll/blog/word/update/star", this.$qs.stringify(data))
          .then(res => {
            if (res.data.result) {
              this.$Message["success"]({
                background: true,
                content: "点赞成功啦！٩(๑>◡<๑)۶ "
              });
            } else {
              this.$Message["error"]({
                background: true,
                content: "点赞失败啦！o(╥﹏╥)o "
              });
            }
          })
          .catch(errr => {
            console.log(error);
          });
      }
    },

    // 弹出回复留言窗口
    handleReply(id, name) {
      this.bid = id;
      this.replyName = name;
      this.imgUrl = this.randomImg(1, 25);
      this.isShowModal = !this.isShowModal;
      this.$nextTick(() => {
        $("#input_conta").html(""); //清除发布框的文本内容
      });
    }
  }
};
</script>
<style lang="scss" scoped>
.word {
  padding: 15px;
  .comments-img > img {
    display: inline-block;
    border-radius: 50%;
    border: 1px solid #eee;
    padding: 3px;
  }
  .show-face-enter-active {
    transition: all 0.25s ease-in;
  }
  .show-face-enter,
  .show-face-leave-to {
    /* transform: scale(0.9); */
    opacity: 0;
    margin-top: -6px;
  }
  .show-face-enter-to,
  .show-face-leave {
    opacity: 1;
    /* transform: scale(1); */
    margin-top: 0px;
  }

  .page {
    display: flex;
    padding: 20px;
    justify-content: center;
    border-top: 1px solid #eee;
  }
  .massage {
    display: flex;
    justify-content: space-between;
    padding: 10px;
    font-family: Georgia;
    border-bottom: 1px solid #eee;
    padding-top: 60px;
    p {
      span {
        color: #d32;
        font-size: 13px;
        font-weight: 600;
        padding-right: 2px;
      }
    }
  }
  .public {
    // margin-top: 30px;
    border-bottom: 1px solid #eee;
    margin-bottom: 25px;
    span {
      font-weight: bold;
      font-size: 16px;
    }
  }
}
.leave-list {
  ul {
    color: #666;
    li {
      margin-right: 20px;
      .list {
        display: flex;
        border-bottom: 1px dashed #eee;
        padding: 15px 10px;
        padding-right: 0px;
        > img {
          width: 35px;
          height: 35px;
          border-radius: 50%;
          border: 1px solid #eee;
          padding: 3px;
        }
        .text-box {
          width: 92%;
          font-size: 13px;
          color: #666;
          margin-left: 12px;
          position: relative;
          .user-name {
            display: flex;
            justify-content: space-between;
            h3.reply-title {
              font-size: 12px;
            }
            h3 {
              font-weight: 400;
              font-size: 16px;
              color: #1abc9c;
              span {
                font-size: 12px;
                color: #666;
                margin: 0 5px;
              }
            }
            div {
              color: #aaa;
              margin-left: 10px;
            }
          }

          .text {
            padding: 5px 0px;
          }
          .repy {
            position: absolute;
            right: 0;
            bottom: -10px;
            cursor: pointer;
            span {
              margin-left: 20px;
              cursor: pointer;
              color: #aaa;
              i {
                margin-right: 5px;
              }
            }
            span:hover {
              color: #33aba0;
            }
          }
          .repy-box {
            > p {
              border-bottom: 1px solid #eee;
              padding: 10px;
            }
          }
        }
      }
      .left {
        margin-left: 30px;
      }
    }
  }
}
.tip {
  text-align: center;
  color: #666;
  padding: 30px;
  border-top: 1px solid #eee;
}
// .textarea::-webkit-input-placeholder {
//   color: #aab2bd;
//   font-size: 12px;
// }
.input-box {
  display: flex;
  padding-bottom: 12px;
  justify-content: space-between;
  label {
    flex-grow: 1;
    margin-right: 30px;
    position: relative;
    i {
      position: absolute;
      right: 10px;
      top: 7px;
      color: #d32;
    }
  }
  label:nth-child(3) {
    margin-right: 0px;
  }
}
.publish_container {
  > p {
    padding: 10px;
    min-height: 100px;
    border-style: none; /*  此步是必须的  */
  }
}
.face-warp {
  position: relative;
  > i {
    display: block;
    width: 20px;
    height: 20px;
    margin-left: 10px;
    cursor: pointer;
  }
  > .i2 {
    background: url("../assets/image/fi1.png") no-repeat;
    background-size: 100%;
  }
  > .i1 {
    background: url("../assets/image/fihover.png") no-repeat;
    background-size: 100%;
  }
  > #face::after {
    content: "";
    position: absolute;
    width: 12px;
    height: 12px;
    top: -7px;
    left: 15px;
    border-color: #e5e5e5;
    border-style: solid;
    border-width: 1px 0 0 1px;
    background: #fff;
    transform: rotate(45deg);
    transition: opacity 0.3s ease-in;
  }
  > #face {
    position: absolute;
    top: 36px;
    left: 0;
    z-index: 10;
    padding: 10px;
    width: 480px;
    border-radius: 5px;
    background-color: #fff;
    border: 1px solid #ddd;
    box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.17);

    transition: all 0.3s ease;
    img {
      float: left;
      margin: 5px;
      cursor: pointer;
      transition: all 0.3s ease;
    }
    img:hover {
      transform: scale(1.3);
    }
  }
}
// .text {
//   img {
//     display: inline-block;
//     width: 25px;
//     width: 25px;
//   }
// }
.comments-box {
  margin: 0px 10px;
  display: flex;
  .comments-img {
    margin-right: 10px;
    img {
      display: block;
      width: 60px;
      height: 60px;
      border-radius: 12px;
    }
  }
  .inner {
    flex-grow: 1;
  }
  .comments-warp {
    width: 100%;
    border: 1px solid #eee;
    border-radius: 5px;

    .comments-textarea {
      width: 100%;
      bottom: 2px;
      font-size: 15px;
      border-top: 1px solid #eee;
      // overflow: hidden;
      display: flex;
      justify-content: space-between;
      align-items: center;
      .submit {
        border: none;
        cursor: pointer;
        text-align: center;
        color: #888;
        padding: 8px 20px;
        border-left: 1px solid #eee;
        // cursor: url("../assets/image/b.png"), auto;
      }
      .submit:hover {
        color: #33aba0;
        // cursor: url("../assets/image/b.png"), auto;
      }
    }
  }
}
.is-reply {
  margin-left: 30px;
}
.inner p {
  text-indent: 0;
}
</style>
