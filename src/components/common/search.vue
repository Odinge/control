<!--
 * @Description: 文章页面
 * @version: 
 * @Author: Lianglin
 * @Date: 2019-08-09 16:48:13
 * @LastEditors: Lianglin
 * @LastEditTime: 2019-09-06 12:39:32
 -->
<template>
  <div class="article">
    <input type="text" v-model="key" class="search" placeholder="请输入文章内容" v-on:keyup="getList()" />
    <Table border :columns="columns12" :data="tabelList">
      <template slot-scope="{ row }">
        <strong>{{ row }}</strong>
      </template>
      <template slot-scope="{ row , index }" slot="action">
        <Button
          type="primary"
          size="small"
          style="margin-right: 5px"
          @click="updateArticle(index)"
        >修改</Button>
        <Button type="error" size="small" @click="remove(index)">刪除</Button>
      </template>
    </Table>
    <Page
      :total="total"
      show-total
      show-sizer
      :current.sync="current"
      :page-size="pagesize"
      @on-change="handleCurrent"
      @on-page-size-change="handlePagesize"
    />
  </div>
</template>
<script>
import Bus from "../../util/eventBus";
import * as api from "../../api/api";
export default {
  data() {
    return {
      columns12: [
        {
          title: "标题",
          key: "title",
          align: "center"
        },
        {
          title: "栏目",
          key: "type",
          align: "center",
          render: (h, params) =>
            h(
              "i-button",
              {
                props: {
                  type: "primary",
                  size: "small"
                },
                style: {
                  marginRight: "5px"
                }
              },
              this.articleType(params.row.type)
            ),
          filters: [
            {
              label: "党内通知",
              value: 0
            },
            {
              label: "党建动态",
              value: 1
            },
            {
              label: "必学要闻",
              value: 2
            },
            {
              label: "热点新闻",
              value: 3
            },
            {
              label: "视频",
              value: 4
            }
          ],
          filterMultiple: false,
          filterMethod: (value, row) => {
            if (value === 0) {
              return row.type == 0;
            } else if (value === 1) {
              return row.type == 1;
            } else if (value === 2) {
              return row.type == 2;
            } else if (value === 3) {
              return row.type == 3;
            } else if(value === 4){
              return row.type == 4;
            }
          }
        },
        {
          title: "作者",
          key: "author",
          align: "center"
        },
        {
          title: "日期",
          key: "updatetime",
          align: "center"
        },
        {
          title: "评论",
          key: "viewCount",
          align: "center"
        },
        {
          title: "操作",
          slot: "action",
          align: "center"
        }
      ],
      tabelList: [],
      total: 0,
      current: 1,
      pagesize: 10,
      key: ""
    };
  },
  created() {
    this.$Spin.show();
    this.getList();
  },
  methods: {
    getList() {
      this.tabelList = [];
      let formData = new FormData();
      formData.append("key", this.key);
      formData.append("page", this.current);
      formData.append("size", this.pagesize);
      setTimeout(() => {
        this.$post(api.searchArticle, formData).then(
          res => {
            if (res.code === 400005) {
              this.tabelList = [];
              this.$Message.info(res.message);
            } else if (res.code === 20001) {
              this.tabelList = res.data.rows;
            }
            this.total = res.data.total;
            setTimeout(() => {
              this.$Spin.hide();
            }, 100);
          }
        );
      }, 100);
    },
    /**
     * @name: 修改文章 post
     * @msg: updateArticle
     * @param {type} 
     * articleId:分页获取文章的时候可以获取到它的id
       author：分页获取文章的时候可以获取
      title:文章标题
      content：文章内容，前端看能不能整合富文本
      （可选）label：文章标签，数据库设计时有的，感觉有点用，但似乎用处不大
      type:文章类型 0:党内通知 1:党建动态 2:必学要闻 3:热点新闻 4:视频
    * @return: 
    */
    updateArticle(index) {
      var row = this.tabelList[index];
      var username = localStorage.getItem("username");
      this.$router.push({ name: "addArticle", params: { row, id: 2 } });
      localStorage.setItem("getArticle", JSON.stringify(row));
    },
    /**
     * @name: 删除文章
     * @msg: deleteArticle
     * @param {type} articleId
     * @return:
     */
    remove(index) {
      var articleId = this.tabelList[index].articleId;
      var row = this.tabelList[index];
      this.$Modal.confirm({
        title: "删除",
        content: `<p>确定删除文章<br>《<span style="font-weight:900">${row.title}</span>》吗？</p>`,
        onOk: () => {
          this.$post(
            api.deleteArticle,
            qs.stringify({
              articleId: articleId
            })
          )
            .then(res => {
              if (res.code == 20001) {
                this.$Message.success(res.message);
                this.getList();
              } else {
                this.$Message.info(res.message);
              }
            })
            .catch(err => {
              this.$Message.error(err.message);
            });
        },
        onCancel: () => {
          this.$Message.info("取消删除");
        }
      });
    },
    handlePagesize(val) {
      this.pagesize = val;
      this.getList();
    },
    handleCurrent(val) {
      this.current = val;
      this.getList();
    },
    /**
     * @name: 表格栏目列数据格式化
     * @msg:
     * @param {type}
     * @return:
     */
    articleType(val) {
      if (val === 0) {
        return "党内通知";
      } else if (val === 1) {
        return "党建动态";
      } else if (val === 2) {
        return "必学要闻";
      } else if (val === 3) {
        return "热点新闻";
      } else {
        return "视频";
      }
    }
  }
};
</script>

