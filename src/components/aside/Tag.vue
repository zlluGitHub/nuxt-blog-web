<template>
  <div class="box-bj-sd">
    <h3 class="h3-style">
      <i class="fa fa-tags"></i>热门标签
    </h3>
    <ul class="tag" v-if="tagsData.length">
      <li v-for="item in tagsData" :key="item.name">
        <!-- <router-link :to="'/detail/'+item.bid">{{item.name}}</router-link> -->
        <!-- <router-link  :to="{ path: '/tags/', query: { id: item.name}}" :style="'background-color:'+ getRandomColor()+';opacity: 0.8;'">
          {{item.name}}
          <span>（{{item.num}}）</span>
        </router-link>-->
        <!-- @click.native="handleTo(item.name,'/tags')" -->
        <a
          :style="'border:1px solid '+ getRandomColor()+';'"
          href="javascript:void(0);"
          @click.stop="handleTo(item.name)"
        >
          {{item.name}}
          <!-- <span>（{{item.num}}）</span> -->
        </a>
      </li>
    </ul>
    <div v-else style="padding:100px 0;text-align: center;">暂无内容！</div>
  </div>
</template>
<script>
// import { goBack } from "../../assets/gloable.js";
export default {
  name: "tag",
  data: () => ({
    tagsData: []
  }),
  // computed: {
  //   geTagsDatas() {
  //     return this.$store.getters.geTagsData;
  //   }
  // },
  // watch: {
  //   geTagsDatas(value) {
  //     this.tagsData = value;
  //   }
  // },
  mounted() {
    let data = this.$store.state.comment.tagData;
    // console.log(this.$store.state.comment);
    
    if (data.length === 0) {
      this.$axios.get(this.$url + "/zll/article/tag").then(res => {
        if (res.data.result) {
          this.tagsData = res.data.list;
          this.$store.commit("setTagData", this.tagsData);
        }
      });
    } else {
      this.tagsData = data;
    }
  },
  methods: {
    // 获取随机颜色
    getRandomColor() {
      return (
        "#" +
        ("00000" + ((Math.random() * 16777215 + 0.5) >> 0).toString(16)).slice(
          -6
        )
      );
    },
    handleTo(name) {
      // this.$store.commit("setTagValue", name);
      this.$router.push({ path: "/tags" });

      this.$store.commit("setTagValue", name);
      // 返回顶部
      this.$goBack();
      // this.$router.push({ path: "/search" });
      // let pageData = this.$store.getters.getTagsData(nav);
      // this.$store.dispatch("setArticleTypeData", pageData);
      // this.$store.dispatch("setRouter", { nav, url, title });
      // this.$store.dispatch("setChangingOver", {
      //   notice: false,
      //   position: true
      // });
    }
  }
};
</script>
<style lang="scss" scoped>
.tag {
  margin-top: 6px;
  display: flex;
  flex-wrap: wrap;
  font-size: 12px;
  padding: 20px;
}

.tag li {
  margin-right: 10px;
  margin-bottom: 12px;
}

.tag li a {
  display: flex;
  padding: 7px 14px;

  font-size: 13px;
  border-radius: 5px;
  box-shadow: 0 3px 5px rgba(0, 0, 0, 0.12);
  color: #666;
  // background-color: #ccc;
  transition: all 0.3s ease;
}

.tag li a:hover {
  opacity: 1 !important;
  transform: scale(1.1);
}
</style>
