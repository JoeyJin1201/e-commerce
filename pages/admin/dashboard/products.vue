<template>
  <v-data-table
    :headers="headers"
    :items="product"
    sort-by="calories"
    class="elevation-1"
  >
    <template #top>
      <v-toolbar flat>
        <v-toolbar-title>My CRUD</v-toolbar-title>
        <v-divider class="mx-4" inset vertical />
        <v-spacer />
        <v-dialog v-model="dialog" max-width="720px">
          <template #activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-file-input
                      accept="image/*"
                      label="上傳圖片"
                    />
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.title" label="標題" />
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.category" label="分類" />
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.unit" label="單位" />
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.origin_price"
                      prefix="$"
                      type="number"
                      label="原價"
                    />
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.price"
                      prefix="$"
                      type="number"
                      label="售價"
                    />
                  </v-col>
                  <v-col cols="12">
                    <v-textarea
                      v-model="editedItem.description"
                      name="input-7-1"
                      filled
                      label="產品描述"
                      auto-grow
                      value="The Woodman set to work at once, and so sharp was his axe that the tree was soon chopped nearly through."
                    />
                  </v-col>
                  <v-col cols="12">
                    <v-textarea
                      v-model="editedItem.content"
                      name="input-7-2"
                      filled
                      label="說明內容"
                      auto-grow
                      value="The Woodman set to work at once, and so sharp was his axe that the tree was soon chopped nearly through."
                    />
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-checkbox
                      v-model="editedItem.is_enabled"
                      label="是否啟用"
                    />
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer />
              <v-btn color="blue darken-1" text @click="close">
                Cancel
              </v-btn>
              <v-btn color="blue darken-1" text @click="save">
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="headline">
              Are you sure you want to delete this item?
            </v-card-title>
            <v-card-actions>
              <v-spacer />
              <v-btn color="blue darken-1" text @click="closeDelete">
                Cancel
              </v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">
                OK
              </v-btn>
              <v-spacer />
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template #[`item.actions`]="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)">
        mdi-pencil
      </v-icon>
      <v-icon small @click="deleteItem(item)">
        mdi-delete
      </v-icon>
    </template>
    <template #no-data>
      <!-- <v-btn color="primary" @click="initialize">
        Reset
      </v-btn> -->
    </template>
  </v-data-table>
</template>

<script>
export default {
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: '分類',
        align: 'start',
        value: 'category'
      },
      { text: '名稱', value: 'title' },
      { text: '原價', value: 'origin_price' },
      { text: '售價', value: 'price' },
      { text: 'Actions', value: 'actions', sortable: false }
    ],
    product: [],
    editedIndex: -1,
    editedItem: {},
    defaultItem: {}
  }),

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    }
  },

  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    }
  },

  created () {
    // this.initialize()
    this.getProducts()
  },

  methods: {
    getProducts (page = 1) {
      const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/products?page=${page}`
      const vm = this
      // vm.isLoading = true
      vm.$axios.get(api).then((response) => {
        // console.log(response.data)
        if (response.data.success) {
          // vm.isLoading = false
          console.log(response.data.products)
          vm.product = response.data.products
        }
      })
    },

    uploadFile () {
      const uploadedFile = this.$refs.files.files[0]
      const vm = this
      const formData = new FormData()
      formData.append('file-to-upload', uploadedFile)
      const url = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/upload`
      vm.status.fileUploading = true
      this.$http.post(url, formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then((response) => {
        vm.status.fileUploading = false
        if (response.data.success) {
          // vm.tempProduct.imageUrl = response.data.imageUrl;
          // console.log(vm.tempProduct);
          vm.$set(vm.tempProduct, 'imageUrl', response.data.imageUrl)
        }
      })
    },

    editItem (item) {
      this.editedIndex = this.product.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem (item) {
      const vm = this
      vm.editedIndex = vm.product.indexOf(item)
      vm.editedItem = Object.assign({}, item)
      vm.dialogDelete = true
      const url = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/product/${vm.editedItem.id}`
      vm.$axios.delete(url).then((response) => {
        // console.log(response, vm.editedItem)
        if (response.data.success) {
          vm.getProducts()
        }
      })
    },

    deleteItemConfirm () {
      this.product.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save () {
      if (this.editedIndex > -1) {
        Object.assign(this.product[this.editedIndex], this.editedItem)
      } else {
        this.product.push(this.editedItem)
      }
      this.close()
    }
  }
}
</script>
