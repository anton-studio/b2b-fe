<template>
  <div>
    <!-- 列表页面 -->
    <div class="container" v-if="!showEdit">
      <div class="header"><div class="title">产品列表</div></div>
      <!-- 表格 -->
        <el-table
        :data="tableData"
        style="width: 100%">
        <el-table-column
          prop="spu_name"
          label="SPU"
          width="150">
        </el-table-column>
        <el-table-column
          prop="spu_title"
          label="商品名称"
          width="150">
        </el-table-column>
        <el-table-column
          label="产品图片"
          width="120">
          <template slot-scope="scope">
            <el-image
            style="width: 100px; height: 100px"
            :src="scope.row.img_url"
            fit="cover"></el-image>
          </template>
        </el-table-column>
        <el-table-column
          prop="catalog_id"
          label="产品分类"
          width="120">
        </el-table-column>
        <el-table-column
          prop="price"
          label="价格区间"
          width="120">
        </el-table-column>
        <el-table-column
        width="120"
        label="是否上架"
        >
        <template slot-scope="props">
              <el-switch
                :value="props.row.publish_status == 1"
                active-color="#3963bc"
              ></el-switch>
            </template>
        </el-table-column>
        <el-table-column
          prop="spu_description"
          label="产品描述"
          width="200">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="Operations"
          width="170">
          <template slot-scope="scope">
            <el-button plain type="primary" size="mini"
                       @click.native.prevent.stop="handleEdit(scope.row.id)"
            >
              Edit
            </el-button>
            <el-button plain type="primary" size="mini"
                       @click.native.prevent.stop="handleDrawer(scope.row.id)"
            >
              Files
            </el-button>
            <el-button plain type="danger" size="mini"
                       @click.native.prevent.stop="handleDelete(scope.row.id)"
            >Delete</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!--file drawer-->
    <el-drawer
      title="产品关联的文件"
      :visible.sync="showDrawer"
      direction="rtl"
      :before-close="handleClose">
      <div>
        <input ref="fileInput" type="file" :disabled="isFileUploading" />
        <button @click="handleUpload" :disabled="isFileUploading">
          {{ isFileUploading ? "uploading" : "upload" }}
        </button>
      </div>
      <el-table
        :data="fileList"
        style="width: 100%">
        <el-table-column
          prop="fileName"
          label="文件名"
          width="180">
        </el-table-column>
        <el-table-column
          label="下载"
          width="180">
          <template slot-scope="scope">
            <a :href="scope.row.fileUrl" target="_blank">downlnoad</a>
          </template>
        </el-table-column>
      </el-table>
    </el-drawer>
  </div>
</template>

<script>
import product from '@/model/product'

export default {
  components: {
  },
  async created() {
    await this.getProducts()
  },
  methods: {
    handleDelete(id) {
      this.$confirm('此操作将永久删除该产品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(async () => {
        const res = await product.deleteProduct(id)
        if (res.code < window.MAX_SUCCESS_CODE) {
          this.getProducts()
          this.$message({
            type: 'success',
            message: `${res.message}`,
          })
        }
      })
    },
    async getProducts() {
      try {
        const products = await product.getProducts()
        this.tableData = products
      } catch (error) {
        if (error.code === 10020) {
          this.tableData = []
        }
      }
    },
    handleEdit(productId) {
      this.$router.push({ path: '/product/edit', query: { id: productId } })
    },
    handleDrawer(productId) {
      this.showDrawer = true
      this.showDrawerForSpuId = productId
      this.getFileList()
      console.log(productId)
    },
    async getFileList() {
      console.log(this.showDrawerForSpuId)
      const res = await product.getFileBySpuId(this.showDrawerForSpuId)
      this.fileList = res.map(item => ({
        fileName: item.file_name,
        fileUrl: item.file_url
      }))
    },
    handleChange() {
      console.log('file list here')
      console.log(this.$refs.upload.fileList)
    },
    handleUpload() {
      const { files } = this.$refs.fileInput
      console.log(files)
      this.sendRequest(files[0])
    },
    sendRequest(file) {
      const self = this
      self.isFileUploading = true
      console.log('sending request')
      return this.$axios({
        method: 'post',
        url: '/cms/file',
        data: {
          [file.name]: file
        },
      })
        .then(async res => {
          console.log('result:', res)
          await product.createFileForSpu({
            spu_id: self.showDrawerForSpuId,
            file_name: res[0].key,
            file_url: res[0].url
          })
          self.isFileUploading = false
          self.getFileList()
        })
    },
  },
  data() {
    return {
      showDrawer: false,
      showDrawerForSpuId: '',
      isFileUploading: false,
      tableColumn: [
        {
          prop: 'spu_name',
          label: 'SPU',
        },
        {
          prop: 'img_url',
          label: '产品图片',
        },
        {
          prop: 'price',
          label: '价格区间',
        },
        {
          prop: 'catalog_id',
          label: '分类ID',
        },
      ],
      tableData: [],
      operate: [
        {
          name: '编辑',
          func: 'handleEdit',
          type: 'primary',
        },
        {
          name: '删除',
          func: 'handleDelete',
          type: 'danger',
          permission: '删除图书',
        },
      ],
      showEdit: false,
      editBookID: 1,
      fileList: [{
        fileName: 'a16 文件',
        fileUrl: 'http://www.baidu.com/a16.zip'
      }, {
        fileName: 'a17 文件',
        fileUrl: 'http://www.baidu.com/a16.zip'
      }, {
        fileName: 'a18 文件',
        fileUrl: 'http://www.baidu.com/a16.zip'
      }]
    }
  }
}

</script>


<style lang="scss" scoped>

.container {
  padding: 0 30px;

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;

    .title {
      height: 59px;
      line-height: 59px;
      color: $parent-title-color;
      font-size: 16px;
      font-weight: 500;
    }
  }

  .pagination {
    display: flex;
    justify-content: flex-end;
    margin: 20px;
  }
}

</style>
