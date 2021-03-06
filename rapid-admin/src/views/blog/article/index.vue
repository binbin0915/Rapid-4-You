<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <el-form v-if="crud.props.searchToggle" :inline="true">
        <!-- 搜索 -->
        <el-form-item>
          <el-input
            v-model="query.blurry"
            clearable
            placeholder="输入标题模糊搜索"
          />
        </el-form-item>
        <el-form-item>
          <el-select
            v-model="query.blogType"
            clearable
            placeholder="按类型搜索"
          >
            <el-option
              v-for="item in blogTypeOptions"
              :key="item.key"
              :label="item.display_name"
              :value="item.key"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select
            v-model="query.categoryId"
            clearable
            placeholder="按分类搜索"
          >
            <el-option
              v-for="item in categoryOptions"
              :key="item.key"
              :label="item.display_name"
              :value="item.key"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select
            v-model="query.archiveId"
            clearable
            placeholder="按归档搜索"
          >
            <el-option
              v-for="item in archiveOptions"
              :key="item.key"
              :label="item.display_name"
              :value="item.key"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select
            v-model="query.tagIds"
            multiple
            clearable
            collapse-tags
            placeholder="按标签搜索"
          >
            <el-option
              v-for="item in tagOptions"
              :key="item.key"
              :label="item.display_name"
              :value="item.key"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <rrOperation />
        </el-form-item>
      </el-form>
      <crudOperation :permission="permission" />
    </div>
    <!--表格渲染-->
    <el-table
      ref="table"
      v-loading="crud.loading"
      :data="crud.data"
      @select="crud.selectChange"
      @select-all="crud.selectAllChange"
      @selection-change="crud.selectionChangeHandler"
    >
      <el-table-column type="selection" width="55" />
      <el-table-column label="标题" width="360" :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <router-link
            :to="{ path: '/blog/article/' + scope.row.id }"
          ><el-link type="primary" style="font-size: unset">{{
            scope.row.title
          }}</el-link></router-link>
        </template>
      </el-table-column>
      <el-table-column label="类型">
        <template slot-scope="scope">
          <template>
            {{ scope.row.blogType | toEnumDesc(blogTypeOptions) }}
          </template>
        </template>
      </el-table-column>
      <el-table-column label="分类" prop="category.name" />
      <el-table-column label="归档" prop="archive.name" />
      <el-table-column label="标签" :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <template>
            {{ scope.row.tags.map((v) => v.name).join(" | ") }}
          </template>
        </template>
      </el-table-column>
      <el-table-column
        v-permission="['admin', 'article:edit', 'article:del']"
        label="操作"
        width="130px"
        align="center"
        fixed="right"
      >
        <template slot-scope="scope">
          <udOperation
            :data="scope.row"
            :permission="permission"
            :disabled-dle="scope.row.id === 1"
            msg="确定删除吗，此操作不能撤销！"
          />
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <pagination />
  </div>
</template>

<script>
import crudArticle from '@/api/blog/article'
import crudCategory from '@/api/blog/category'
import crudArchive from '@/api/blog/archive'
import crudTag from '@/api/blog/tag'
import CRUD, { presenter, header, form, crud } from '@/components/Crud/crud'
import rrOperation from '@/components/Crud/RR.operation'
import crudOperation from '@/components/Crud/CRUD.operation'
import udOperation from '@/components/Crud/UD.operation'
import pagination from '@/components/Crud/Pagination'

const defaultForm = {
  id: null,
  name: null,
  isTop: true,
  subCount: 0,
  pid: null,
  deptSort: 999,
  enabled: 'true'
}
export default {
  name: 'ArticleList',
  components: {
    crudOperation,
    rrOperation,
    udOperation,
    pagination
  },
  cruds() {
    return CRUD({
      title: '文章',
      url: 'api/articles',
      showOnPage: true,
      crudMethod: { ...crudArticle },
      sort: ['createTime,desc']
    })
  },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  // 设置数据字典
  dicts: [],
  data() {
    return {
      blogTypeOptions: [
        { key: 'NORMAL', display_name: '普通' },
        { key: 'SUBJECT', display_name: '专题' },
        { key: 'ABOUT', display_name: '关于' }
      ],
      categoryOptions: [],
      archiveOptions: [],
      tagOptions: [],
      articles: [],
      rules: {},
      permission: {
        add: ['admin', 'article:add'],
        edit: ['admin', 'article:edit'],
        del: ['admin', 'article:del']
      }
    }
  },
  created() {
    this.crud.optShow.download = false

    // 获取分类下拉
    crudCategory.getAll().then((res) => {
      this.categoryOptions = res.map(function(v) {
        return { key: v.id, display_name: v.name }
      })
    })
    // 获取归档下拉
    crudArchive.getAll().then((res) => {
      this.archiveOptions = res.map(function(v) {
        return { key: v.id, display_name: v.name }
      })
    })
    // 获取标签下拉
    crudTag.getAll().then((res) => {
      this.tagOptions = res.map(function(v) {
        return { key: v.id, display_name: v.name }
      })
    })
  },
  methods: {
    // 新建
    [CRUD.HOOK.beforeToAdd](crud) {
      this.$router.push({ name: 'ArticleAdd' })
    },
    // 编辑
    [CRUD.HOOK.beforeToEdit](crud, article) {
      this.$router.push('/blog/article/edit/' + article.id)
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
</style>
