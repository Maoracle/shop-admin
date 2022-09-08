<template>
  <page-container>
    <app-card>
      <template #header>
        数据筛选
      </template>
      <el-form
        :inline="true"
        ref="form"
        :model="listParams"
        :disabled="listLoading"
        @submit.prevent="handleQuery"
      >
        <el-form-item label="商品分类">
          <el-select
            v-model="listParams.pid"
            placeholder="请选择"
            clearable
            @change="loadList"
          >
            <el-option
              label="全部"
              :value="0"
            />
            <el-option
              v-for="item in productCates"
              :key="item.id"
              :label="`${item.html}${item.cate_name}`"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="状态">
          <el-select
            v-model="listParams.is_show"
            placeholder="请选择"
            clearable
          >
            <el-option
              label="显示"
              :value="1"
            />
            <el-option
              label="隐藏"
              :value="0"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="分类名称">
          <el-input
            v-model="listParams.cate_name"
            clearable
            placeholder="请输入"
          />
        </el-form-item>
        <el-form-item>
          <el-button native-type="submit">
            查询
          </el-button>
        </el-form-item>
      </el-form>
    </app-card>
    <app-card>
      <template #header>
        <el-button
          type="primary"
          @click="formVisible = true"
        >
          添加分类
        </el-button>
      </template>
      <!--
        启用树菜单：
          1. data 数据需要是树结构
          2. 给 vxe-table 组件设置 row-id
          3. 给 vxe-column 设置 tree-node
       -->
      <vxe-table
        :data="list"
        row-id="id"
        :tree-config="{ children: 'children' }"
        v-loading="listLoading"
      >
        <vxe-column
          field="id"
          title="ID"
        />
        <vxe-column
          field="cate_name"
          title="分类名称"
          tree-node
        />
        <vxe-column
          field="pic"
          title="分类图标"
        />
        <vxe-column
          field="sort"
          title="排序"
        />
        <vxe-column title="状态">
          <template #default="{ row }">
            <el-switch
              v-model="row.is_show"
              active-color="#13ce66"
              inactive-color="#ff4949"
              :active-value="1"
              :inactive-value="0"
              :loading="row.statusLoading"
              @change="handleStatusChange(row)"
            />
          </template>
        </vxe-column>
        <vxe-column
          title="操作"
          min-width="100"
        >
          <template #default="scope">
            <el-button
              type="text"
              @click="handleUpdate(scope.row.id)"
            >
              编辑
            </el-button>
            <el-popconfirm
              title="确认删除吗？"
              @confirm="handleDelete(scope.row.id)"
            >
              <template #reference>
                <el-button type="text">
                  删除
                </el-button>
              </template>
            </el-popconfirm>
          </template>
        </vxe-column>
      </vxe-table>
    </app-card>
  </page-container>
  <class-form
    v-model="formVisible"
    v-model:role-id="roleId"
    @success="handleFormSuccess"
  />
</template>

<script lang="ts" setup>
import { ref, reactive, onMounted } from 'vue'
import { deleteRole } from '@/api/role'
import * as productApi from '@/api/product'
import type { ProductCategory, ProductCategoryListParams, ProductCategoryList } from '@/api/types/product'
import { ElMessage } from 'element-plus'
import ClassForm from './ClassForm.vue'

const list = ref<ProductCategoryList[]>([]) // 列表数据
const productCates = ref<ProductCategory[]>([])
const listCount = ref(0)
const listLoading = ref(true)
const listParams = reactive<ProductCategoryListParams>({ // 列表数据查询参数
  page: 1, // 当前页码
  limit: 10, // 每页大小
  is_show: null,
  pid: null,
  cate_name: ''
})
const formVisible = ref(false)
const roleId = ref<number | null>(null)

onMounted(() => {
  loadList()
  loadProductCates()
})

const loadList = async () => {
  listLoading.value = true
  const data = await productApi.getProductsCategory(listParams).finally(() => {
    listLoading.value = false
  })
  data.list.forEach(item => {
    item.statusLoading = false // 控制切换状态的 loading 效果
  })
  list.value = data.list
  listCount.value = data.count
}

const loadProductCates = async () => {
  const data = await productApi.getCategoryTree(0)
  productCates.value = data
}

const handleQuery = async () => {
  listParams.page = 1 // 查询默认从第1页开始
  loadList()
}

const handleDelete = async (id: number) => {
  await deleteRole(id)
  ElMessage.success('删除成功')
  loadList()
}

const handleStatusChange = async (item: ProductCategoryList) => {
  item.statusLoading = true
  await productApi.updateProductCategoryStatus(item.id, item.is_show).finally(() => {
    item.statusLoading = false
  })
  ElMessage.success(`${item.is_show === 1 ? '显示' : '隐藏'}成功`)
}

const handleUpdate = (id: number) => {
  roleId.value = id
  formVisible.value = true
}

const handleFormSuccess = () => {
  formVisible.value = false
  loadList()
}

</script>

<style lang="scss" scoped>
.text-nowrap {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
</style>
