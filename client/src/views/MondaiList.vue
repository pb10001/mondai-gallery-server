<template>
  <div>
      <div class="section has-background-white mb">
        <div class="media">
          <div class="media-content">
            <div class="content">
              <div>
                <p class="title is-3">{{list.mondaiList.name}}</p>
                <p class="subtitle is-4 mb">
                  <img
                    :src="list.mondaiList.editor.picUrl"
                    v-if="list.mondaiList.editor.picUrl"
                    width="32"
                    height="32"
                    alt="No Image"
                  >
                  <b-icon
                    v-else
                    icon="account-box"
                    :style="{'color': list.mondaiList.editor.color||'#555'}"
                  />
                  <a @click="to(profileUrl())">{{list.mondaiList.editor.nickname}}</a>
                </p>
              </div>
            </div>
          </div>
        </div>
        <div class="content">
          <b-taglist>
            <b-tag class="is-primary" v-if="list.mondaiList.fromMyMondais">自作問題</b-tag>
            <b-tag v-for="tag in list.mondaiList.tags" :key="tag">{{tag}}</b-tag>
          </b-taglist>
          <div v-if="!isRead()" class="buttons has-addons">
            <span class="button is-light is-small" @click="read">
              <b-icon size="is-small" icon="check"/>
              <span>読んだ！</span>
            </span>
            <span
              v-if="list.mondaiList.read.length"
              class="button is-info is-small"
            >{{list.mondaiList.read.length}}</span>
          </div>
          <div v-else class="buttons has-addons">
            <span class="button is-primary is-small" @click="unread">
              <b-icon size="is-small" icon="check"/>
              <span>読んだ！済</span>
            </span>
            <span
              v-if="list.mondaiList.read.length"
              class="button is-info is-small"
            >{{list.mondaiList.read.length}}</span>
          </div>
        </div>
        <div id="markdown" class="content" v-html="renderMarkdown(list.mondaiList.description)"></div>
        <footer v-if="list.isMine">
          <a class="" @click="$router.push(editUrl())">
            <b-icon icon="pencil"></b-icon>
            <span>編集</span>
          </a>
        </footer>
      </div>
    <div class="columns">
      <div class="column is-one-third">
        <b-collapse class="card">
          <header slot="trigger" slot-scope="props" class="card-header">
            <p class="card-header-title">統計</p>
            <a class="card-header-icon">
              <b-icon :icon="props.open ? 'menu-down' : 'menu-up'"></b-icon>
            </a>
          </header>
          <div class="card-content">
            <div class="columns is-mobile is-multiline" v-if="list.mondaiList.mondai.length > 0">
              <div class="column is-6">
                <SiteChart :list="list.mondaiList.mondai"/>
              </div>
              <div class="column is-6">
                <AuthorChart :list="list.mondaiList.mondai"/>
              </div>
              <div class="column is-12">
                <GenreChart :list="list.mondaiList.mondai"/>
              </div>
            </div>
          </div>
        </b-collapse>
      </div>
      <div class="column is-two-thirds">
        <b-collapse class="card mb" :open="false">
          <div slot="trigger" slot-scope="props" class="card-header">
            <p class="card-header-title">フィルター</p>
            <a class="card-header-icon">
              <b-icon :icon="props.open ? 'menu-down' : 'menu-up'"></b-icon>
            </a>
          </div>
          <div class="card-content">
            <div class="content">
              <div class="level">
                <div class="level-left">
                  <div class="level-item">
                    <b-field label="サイト">
                      <b-select placeholder="Select a name" v-model="siteFilter">
                        <option
                          v-for="item in siteList"
                          :value="item.key"
                          :key="item.key"
                        >{{item.value.name}}</option>
                      </b-select>
                    </b-field>
                  </div>
                  <div class="level-item">
                    <b-field label="ジャンル">
                      <b-select placeholder="Select a genre" v-model="genreFilter">
                        <option
                          v-for="item in genreList"
                          :value="item.key"
                          :key="item.key"
                        >{{item.value}}</option>
                      </b-select>
                    </b-field>
                  </div>
                </div>
              </div>
              <div class="level">
                <div class="level-left">
                  <a class="button is-outlined is-danger" v-on:click="clearFilter()">クリア</a>
                </div>
              </div>
              <div class="level">
                <b-field label="表示">
                  <b-select v-model="detail">
                    <option :value="false">リスト</option>
                    <option :value="true">詳細</option>
                  </b-select>
                </b-field>
              </div>
            </div>
          </div>
        </b-collapse>
        <!-- リスト -->
        <div class="panel">
          <transition-group v-if="!detail" name="mondai">
            <a
              class="panel-block"
              v-for="item in filter(list.mondaiList.mondai)"
              v-bind:key="item._id"
              target="_blank"
              v-bind:href="url(item.site,item.id)"
              :data-index="item._id"
            >
              <simple-mondai :item="item"/>
            </a>
          </transition-group>
          <div v-else class="columns is-multiline is-gapless">
            <div
              class="column is-one-third"
              v-for="item in filter(list.mondaiList.mondai)"
              v-bind:key="item._id"
              target="_blank"
              :data-index="item._id"
            >
              <MondaiCard :item="item"></MondaiCard>
            </div>
          </div>
        </div>
        <a 
          v-if="list.mondaiList.addible"
          class="button is-info mb"
          @click="isAddMondaiModalActive = true">問題を追加</a>
        <b-modal id="myModal" :active.sync="isAddMondaiModalActive" has-modal-card>
          <div class="modal-card">
            <div class="modal-card-body">
              <mondai-dialog :mondai="newMondai"></mondai-dialog>
              <button class="button" type="button" @click="isAddMondaiModalActive = false">キャンセル</button>
              <button class="button is-primary" @click="handleOk">追加</button>
            </div>
          </div>
        </b-modal>
        <!-- 次に見る -->
        <div class="panel" v-if="list.otherList.length > 0">
          <div class="panel-heading">
            <b-icon icon="arrow-right-drop-circle-outline"></b-icon>
            <span>次に見る</span>
          </div>
          <SimpleListLinkView :item="item" v-for="item in list.otherList" :key="item.id"/>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import SimpleMondaiView from "@/components/SimpleMondaiView";
import SimpleListLinkView from "@/components/SimpleListLinkView";
import MondaiCard from "@/components/MondaiCard";

import SiteChart from "@/components/SiteChart.vue";
import AuthorChart from "@/components/AuthorChart.vue";
import GenreChart from "@/components/GenreChart.vue";

import marked from "@/markdown.js"
export default {
  mixins: [marked],
  components: {
    "simple-mondai": SimpleMondaiView,
    SimpleListLinkView,
    MondaiCard,
    SiteChart,
    AuthorChart,
    GenreChart
  },
  data() {
    return {
      name: "MondaiList",
      genreFilter: "all",
      siteFilter: "all",
      detail: false,
      isMine: false,
      otherList: [],
      isAddMondaiModalActive: false,
      newMondai: {
        id: 0,
        title: "",
        author: "",
        site: "latethink",
        description: "",
        genre: "umigame",
        _id: -1
      }
    };
  },
  computed: {
    picture() {
      return this.$store.state.picture;
    },
    site: function() {
      return this.$store.state.site;
    },
    siteList: function() {
      let list = [{ key: "all", value: { name: "すべてのサイト" } }];
      for (let key in this.$store.state.site) {
        list.push({ key: key, value: this.$store.state.site[key] });
      }
      return list;
    },
    genre: function() {
      return this.$store.state.genre;
    },
    genreList: function() {
      let list = [{ key: "all", value: "すべてのジャンル" }];
      for (let key in this.$store.state.genre) {
        list.push({ key: key, value: this.$store.state.genre[key] });
      }
      return list;
    },
    list() {
      let id = this.$route.params.id;
      return this.$store.state.savedLists[id];
    }
  },
  watch: {
    $route() {
      this.fetchList();
    }
  },
  created: function() {
    if (!this.list) {
      console.log("Fetch the list from the server");
      this.fetchList();
    } else console.log("Retrieve the saved list");
  },
  methods: {
    url: function(siteName, id) {
      return this.site[siteName].showUrl + id;
    },
    editUrl: function() {
      return "/mondaiList/edit/" + this.list.mondaiList.id;
    },
    profileUrl: function() {
      return "/profile/show/" + this.list.mondaiList.editor.id;
    },
    to: function(url) {
      this.$router.push(url);
    },
    filter: function(mondai) {
      let filtered = mondai;
      // ジャンル
      if (this.genreFilter !== "all")
        filtered = filtered.filter(x => x.genre === this.genreFilter);
      // サイト
      if (this.siteFilter !== "all")
        filtered = filtered.filter(x => x.site === this.siteFilter);

      return filtered;
    },
    sort: function(mondai) {
      let sorted = mondai.sort((x, y) => {
        return x._id - y._id;
      });
      return sorted;
    },
    clearFilter: function() {
      this.genreFilter = "all";
      this.siteFilter = "all";
    },
    fetchList: function() {
      let id = this.$route.params.id;
      this.$store.dispatch("fetchList", id);
    },
    read() {
      if (!this.list.mondaiList) return;
      this.$http
        .post(this.$endPoint("/api/read/" + this.$route.params.id), {
          id: this.$route.params.id,
          editor: this.list.mondaiList.editor || { username: "" }
        })
        .then(res => {
          if (res.data.message === "success") {
            this.fetchList();
            this.$toast.open({
              type: "is-success",
              message: "読んだ！しました。"
            });
          } else {
            this.$toast.open({
              type: "is-danger",
              message: "ログインが必要です。"
            });
          }
        })
        .catch(() => {
          this.$toast.open({
            type: "is-danger",
            message: "ログインが必要です。"
          });
        });
    },
    unread() {
      this.$http
        .post(this.$endPoint("/api/unread/" + this.$route.params.id), {
          id: this.$route.params.id,
          editor: this.list.mondaiList.editor || { username: "" }
        })
        .then(res => {
          if (res.data.message === "success") {
            this.fetchList();
            this.$toast.open({
              type: "is-success",
              message: "読んだ！を解除しました。"
            });
          } else {
            this.$toast.open({
              type: "is-danger",
              message: "ログインが必要です。"
            });
          }
        })
        .catch(() => {
          this.$toast.open({
            type: "is-danger",
            message: "ログインが必要です。"
          });
        });
    },
    isRead() {
      if (this.list.mondaiList)
        return this.list.mondaiList.read.includes(this.$store.state.user.id);
      else return false;
    },
    handleOk() {
      let obj = Object.assign({}, this.newMondai);
      this.$http
        .post(this.$endPoint("/api/mondaiList/" + this.$route.params.id + "/addmondai"), obj)
        .then(() => {
            this.fetchList();
            this.$toast.open({
              type: "is-success",
              message: "問題を追加しました。"
            });
        })
        .catch(() => {
          this.$toast.open({
            type: "is-danger",
            message: "ログインが必要です。"
          });
        });
    }
  }
};
</script>
<style scoped>
.level {
  width: 100%;
}
.mr {
  margin-right: 5px;
}
.mb {
  margin-bottom: 10px;
}
small {
  margin-right: 5px;
}
.mondai-move {
  transition: all 1s ease;
}
.mondai-enter-active {
  transition: all 1s ease;
  transition-delay: 0.3s;
}
.mondai-enter {
  transform: rotateX(360deg);
  opacity: 0;
}
.mondai-leave-active {
  display: none;
  position: absolute;
}
.mondai-leave-to {
  opacity: 0;
}
</style>
