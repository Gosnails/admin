<template>
  <div class="article">
    <el-top :keywords="keywords" @onSearch="handleSearch">
      <el-button @click="handleAddArticle">添加文章</el-button>
    </el-top>
    <el-table v-loading="loading" :data="tableData" border style="width: 100%">
      <el-table-column prop="title" label="标题" />
      <el-table-column prop="type" label="类型" />
      <el-table-column prop="createdAt" label="时间" />
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<script>
import ElTop from "@/components/Top";
import { formatTime } from "@/utils";
import { getClassifyList } from "@/api/classify";
import { getArticleList, deleteById } from "@/api/article";

export default {
  components: { ElTop },
  data() {
    return {
      keywords: "",
      tableData: [],
      classifyData: [],
      loading: false
    };
  },
  created() {
    getClassifyList().then(res => {
      this.classifyData = res.data;
      this._getArticleList();
    })
  },
  methods: {
    _getArticleList() {
      getArticleList({per_page: 50}).then(res => {
        this.tableData = res.data;
        for (let i = 0; i < this.tableData.length; i++) {
          const filterData = this.classifyData.filter(item => item._id === this.tableData[i].type)[0]
          this.tableData[i].type = filterData ? filterData.name : '';
          this.tableData[i].createdAt = formatTime(
            new Date(this.tableData[i].createdAt).getTime(),
            "{y}-{m}-{d} {h}:{i}:{s}"
          );
        }
      });
    },
    handleSearch() {},
    handleAddArticle() {
      this.$router.push({
        path: `/article/add`
      });
    },
    handleEdit(index, item) {
      this.$router.push({
        path: `/article/edit/${item._id}`
      });
    },
    handleDelete(index, item) {
      this.$confirm("此操作将永久删除该文章, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.loading = true;
          deleteById(item._id)
            .then(res => {
              this.loading = false;
              this.$message({
                type: "success",
                message: "删除成功!"
              });
              this._getArticleList();
            })
            .catch(() => {
              this.loading = false;
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>
<style lang="scss" scoped>
.article {
  padding: 25px;
}
</style>
