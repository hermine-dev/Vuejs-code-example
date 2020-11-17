<template>
  <div>
    <app-breadcrumb title="Users"></app-breadcrumb>
    <app-card>
      <div class="app-card-header primary-actions">
        <div class="heading-block">
          <h4 class="heading">Users</h4>
        </div>
        <div class="actions-block">
          <v-btn
            class="i-btn-hoverable"
            color="primary"
            flat
            small
            :ripple="false"
            @click="addUser.dialog = true"
          >
            Add new user<v-icon class="ml-1">add</v-icon>
          </v-btn>
          <v-btn
            :disabled="users.selected.length < 1"
            color="error"
            flat
            small
            :ripple="false"
            @click="deleteUser($event, users.selected, 'group')"
          >
            <v-icon>delete</v-icon>
          </v-btn>
        </div>
      </div>
      <div class="card-body pa-0">
        <v-progress-linear
          v-if="getLoader"
          :indeterminate="true"
          height="3"
          color="primary"
          class="progress-card" />
        <v-data-table
          v-if="!getNoData"
          v-model="users.selected"
          :items="getUsers.users"
          :headers="users.headers"
          :rows-per-page-items="users.rowsPerPageItems"
          :pagination.sync="users.pagination"
          item-key="user_id"
          select-all
          class="i-table"
        >
          <template v-slot:items="props">
            <tr>
              <td class="dropdown-filter-primary align-center">
                <v-checkbox
                  v-model="props.selected"
                  color="primary"
                  :ripple="false"
                  hide-details
                ></v-checkbox>
              </td>
              <td>{{ props.item.business_title }}</td>
              <td>{{ props.item.full_name }}</td>
              <td>{{ props.item.username }}</td>
              <td>
                <template v-for="(item, index) in props.item.categories">
                  {{ index !== props.item.categories.length - 1 ? `${item}, ` : item }}
                </template>
              </td>
              <td>
                <template v-for="(item, index) in props.item.brands">
                  {{ index !== props.item.brands.length - 1 ? `${item}, ` : item }}
                </template>
              </td>
              <td class="td-sm">
                <v-btn
                  class="ma-0 pa-0 btn-table-action"
                  color="primary"
                  flat
                  small
                  :ripple="false"
                  @click="editUser($event, props.item.user_id)"
                >
                  <v-icon>edit</v-icon>
                </v-btn>
                <v-btn
                  class="ma-0 pa-0 btn-table-action"
                  color="error"
                  flat
                  small
                  :ripple="false"
                  @click="deleteUser($event, props.item.user_id, 'one')"
                >
                  <v-icon>delete</v-icon>
                </v-btn>
              </td>
            </tr>
          </template>
        </v-data-table>
        <i-no-data heading="Oops!" v-if="getNoData" />
      </div>
    </app-card>

    <!--Add new User Dialog-->
    <v-dialog
      v-model="addUser.dialog"
      transition="slide-x-transition"
      content-class="i-dialog"
      width="500"
      :dark="getThemeColor === '0'"
      attach
    >
      <v-card>
        <v-card-title
          class="headline"
          primary-title
        >
          Add New User
          <v-btn
            class="modal-close-btn"
            :ripple="false"
            small
            flat
            solo
            @click="addUser.dialog = false"
          >
            <v-icon>close</v-icon>
          </v-btn>
        </v-card-title>
        <v-form
          ref="addUserForm"
          v-model="addUser.valid"
          @submit.prevent="saveUser()"
          lazy-validation
        >
          <v-card-text class="dialog-dropdowns-list">
            <v-layout>
              <v-flex xs6>
                <p>Business title</p>
              </v-flex>
              <v-flex xs6>
                <v-text-field
                  v-model="addUser.businessTitle.model"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Business title"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                >
                </v-text-field>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Full name</p>
              </v-flex>
              <v-flex xs6>
                <v-text-field
                  v-model="addUser.fullName.model"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Full Name"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                >
                </v-text-field>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Categories</p>
              </v-flex>
              <v-flex xs6>
                <div class="dropdown-filter-secondary chip-holder">
                  <v-autocomplete
                    v-model="addUser.categories.model"
                    :items="getNewUserCategories"
                    :loading="addUser.categories.isLoading"
                    :search-input.sync="addUser.categories.search"
                    :ripple="false"
                    :rules="[v => !!v || 'Item is required']"
                    attach
                    background-color="rgba(235, 237, 242, 0.5)"
                    hide-details
                    hide-selected
                    clear-icon
                    multiple
                    cache-items
                    append-icon="expand_more"
                    item-text="name"
                    item-value="id"
                    label="Categories"
                    solo
                    flat
                  >
                    <template v-slot:item="{ item }">
                      <v-list-tile-content>
                        <v-list-tile-title :class="`option-level-${item.level}`" v-text="item.name"></v-list-tile-title>
                      </v-list-tile-content>
                    </template>
                    <template v-slot:selection="data">
                      <div class="i-v-chip">
                        {{ data.item.name }} <v-icon @click="removeChip(data.item, 'category')">close</v-icon>
                      </div>
                    </template>
                  </v-autocomplete>
                </div>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Brands</p>
              </v-flex>
              <v-flex xs6>
                <div class="dropdown-filter-secondary chip-holder">
                  <v-autocomplete
                    v-model="addUser.brands.model"
                    :items="getNewUserBrands"
                    :loading="addUser.brands.isLoading"
                    :search-input.sync="addUser.brands.search"
                    :ripple="false"
                    attach
                    :rules="[v => !!v || 'Item is required']"
                    background-color="rgba(235, 237, 242, 0.5)"
                    hide-details
                    hide-selected
                    clear-icon
                    multiple
                    cache-items
                    chips
                    append-icon="expand_more"
                    item-text="name"
                    item-value="id"
                    label="Brands"
                    solo
                    flat
                  >
                    <template v-slot:item="{ item }">
                      <v-list-tile-content>
                        <v-list-tile-title :class="`option-level-${item.level}`" v-text="item.name"></v-list-tile-title>
                      </v-list-tile-content>
                    </template>
                    <template v-slot:selection="data">
                      <div class="i-v-chip">
                        {{ data.item.name }} <v-icon @click="removeChip(data.item, 'brand')">close</v-icon>
                      </div>
                    </template>
                  </v-autocomplete>
                </div>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Username</p>
              </v-flex>
              <v-flex xs6>
                <v-text-field
                  v-model="addUser.username.model"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Username"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                >
                </v-text-field>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Password</p>
              </v-flex>
              <v-flex xs6>
                <v-text-field
                  v-model="addUser.password.model"
                  type="password"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Password"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                >
                </v-text-field>
              </v-flex>
            </v-layout>
            <div class="text-xs-right">
              <v-btn
                type="submit"
                class="i-btn-hoverable text-normal mx-0 mt-3"
                flat
                color="primary"
                :ripple="false"
              >
                Save
              </v-btn>
            </div>
          </v-card-text>
        </v-form>
      </v-card>
    </v-dialog>

    <!--Edit User Dialog-->
    <v-dialog
      v-model="editUserInfo.dialog"
      transition="slide-x-transition"
      content-class="i-dialog"
      width="500"
      :dark="getThemeColor === '0'"
      attach
    >
      <v-card>
        <v-card-title
          class="headline"
          primary-title
        >
          Edit User {{editUserInfo.id}}
          <v-btn
            class="modal-close-btn"
            :ripple="false"
            small
            flat
            solo
            @click="editUserInfo.dialog = false"
          >
            <v-icon>close</v-icon>
          </v-btn>
        </v-card-title>
        <v-form
          ref="editUserForm"
          v-model="editUserInfo.valid"
          @submit.prevent="saveEditedUser()"
          lazy-validation
        >
          <v-card-text class="dialog-dropdowns-list">
            <v-layout>
              <v-flex xs6>
                <p>Business title</p>
              </v-flex>
              <v-flex xs6>
                <v-text-field
                  :value="getUser.business_title"
                  @input="updateBusinessTitle($event)"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Business title"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                >
                </v-text-field>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Full name</p>
              </v-flex>
              <v-flex xs6>
                <v-text-field
                  :value="getUser.full_name"
                  @input="updateFullName($event)"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Full Name"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                >
                </v-text-field>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Categories</p>
              </v-flex>
              <v-flex xs6>
                <div class="dropdown-filter-secondary chip-holder">
                  <v-autocomplete
                    v-model="editUserInfo.categories.model"
                    :items="getNewUserCategories"
                    :loading="editUserInfo.categories.isLoading"
                    :search-input.sync="editUserInfo.categories.search"
                    :ripple="false"
                    attach
                    :rules="[v => !!v || 'Item is required']"
                    background-color="rgba(235, 237, 242, 0.5)"
                    hide-details
                    hide-selected
                    clear-icon
                    multiple
                    cache-items
                    append-icon="expand_more"
                    item-text="name"
                    item-value="id"
                    label="Categories"
                    solo
                    flat
                  >
                    <template v-slot:item="{ item }">
                      <v-list-tile-content>
                        <v-list-tile-title :class="`option-level-${item.level}`" v-text="item.name"></v-list-tile-title>
                      </v-list-tile-content>
                    </template>
                    <template v-slot:selection="data">
                      <div class="i-v-chip">
                        {{ data.item.name }} <v-icon @click="removeChip(data.item, 'category', 'edit')">close</v-icon>
                      </div>
                    </template>
                  </v-autocomplete>
                </div>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Brands</p>
              </v-flex>
              <v-flex xs6>
                <div class="dropdown-filter-secondary chip-holder">
                  <v-autocomplete
                    v-model="editUserInfo.brands.model"
                    :items="getNewUserBrands"
                    :loading="editUserInfo.brands.isLoading"
                    :search-input.sync="editUserInfo.brands.search"
                    :ripple="false"
                    attach
                    :rules="[v => !!v || 'Item is required']"
                    background-color="rgba(235, 237, 242, 0.5)"
                    hide-details
                    hide-selected
                    clear-icon
                    multiple
                    cache-items
                    chips
                    append-icon="expand_more"
                    item-text="name"
                    item-value="id"
                    label="Brands"
                    solo
                    flat
                  >
                    <template v-slot:item="{ item }">
                      <v-list-tile-content>
                        <v-list-tile-title :class="`option-level-${item.level}`" v-text="item.name"></v-list-tile-title>
                      </v-list-tile-content>
                    </template>
                    <template v-slot:selection="data">
                      <div class="i-v-chip">
                        {{ data.item.name }} <v-icon @click="removeChip(data.item, 'brand', 'edit')">close</v-icon>
                      </div>
                    </template>
                  </v-autocomplete>
                </div>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs6>
                <p>Username</p>
              </v-flex>
              <v-flex xs6>
                <v-text-field
                  :value="getUser.username"
                  @input="updateUsername($event)"
                  class="i-v-text-field ma-0"
                  solo
                  flat
                  hide-details
                  placeholder="Username"
                  :rules="[v => !!v || 'Item is required']"
                  validate-on-blur
                >
                </v-text-field>
              </v-flex>
            </v-layout>
            <div class="text-xs-right">
              <v-btn
                type="submit"
                class="i-btn-hoverable text-normal mx-0 mt-3"
                flat
                color="primary"
                :ripple="false"
              >
                Save
              </v-btn>
            </div>
          </v-card-text>
        </v-form>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  name: 'Users',
  data: () => ({
    test: [0, 4],
    users: {
      headers: [
        { text: 'Business title', value: null, sortable: false },
        { text: 'Full name', value: null, sortable: false },
        { text: 'Username', value: null, sortable: false },
        { text: 'Categories', value: null, sortable: false },
        { text: 'Brands', value: null, sortable: false },
        { text: null, value: null, sortable: false }
      ],
      rowsPerPageItems: [10, 25, 50, 100],
      pagination: {
        rowsPerPage: 25
      },
      selected: []
    },
    addUser: {
      dialog: false,
      valid: true,
      businessTitle: {
        model: ''
      },
      fullName: {
        model: ''
      },
      categories: {
        isLoading: false,
        items: [],
        model: null,
        search: null
      },
      brands: {
        isLoading: false,
        items: [],
        model: null,
        search: null
      },
      username: {
        model: ''
      },
      password: {
        model: ''
      }
    },
    editUserInfo: {
      dialog: false,
      valid: true,
      id: null,
      categories: {
        model: [],
        names: [],
        search: '',
        isLoading: false
      },
      brands: {
        model: [],
        names: [],
        search: '',
        isLoading: false
      }
    },
    timer: 3000
  }),
  methods: {
    removeChip (item, dropdown, modalType) {
      if (modalType !== 'edit') {
        if (dropdown === 'category') {
          const index = this.addUser.categories.model.indexOf(item.id)
          if (index >= 0) this.addUser.categories.model.splice(index, 1)
        } else if (dropdown === 'brand') {
          const index = this.addUser.brands.model.indexOf(item.id)
          if (index >= 0) this.addUser.brands.model.splice(index, 1)
        }
      } else if (modalType === 'edit') {
        if (dropdown === 'category') {
          const index = this.editUserInfo.categories.model.indexOf(item.id)
          if (index >= 0) this.editUserInfo.categories.model.splice(index, 1)
        } else if (dropdown === 'brand') {
          const index = this.editUserInfo.brands.model.indexOf(item.id)
          if (index >= 0) this.editUserInfo.brands.model.splice(index, 1)
        }
      }
    },
    saveUser () {
      if (this.$refs.addUserForm.validate()) {
        this.snackbar = true
      }
      if (this.addUser.valid === true) {
        this.$store.dispatch('usersModule/saveUser', {
          username: this.addUser.username.model,
          password: this.addUser.password.model,
          businessTitle: this.addUser.businessTitle.model,
          fullName: this.addUser.fullName.model,
          categories: this.addUser.categories.model,
          brands: this.addUser.brands.model,
          userId: null
        }).then(() => {
          this.addUser.dialog = false
          this.$refs.addUserForm.reset()
          this.$store.dispatch('usersModule/getUsers')
        })
      }
    },
    saveEditedUser () {
      if (this.$refs.editUserForm.validate()) {
        this.snackbar = true
      }
      if (this.editUserInfo.valid === true) {
        this.$store.dispatch('usersModule/saveUser', {
          username: this.getUser.username,
          password: '',
          businessTitle: this.getUser.business_title,
          fullName: this.getUser.full_name,
          categories: this.editUserInfo.categories.model,
          brands: this.editUserInfo.brands.model,
          userId: this.editUserInfo.id
        }).then(() => {
          this.editUserInfo.dialog = false
          this.$refs.editUserForm.reset()
          this.$store.dispatch('usersModule/getUsers')
        })
      }
    },
    deleteUser (e, items, type) {
      this.$store.dispatch('usersModule/deleteUser', {
        userIds: type === 'one' ? [items] : this.users.selected.map(e => e.user_id)
      }).then(() => {
        this.$store.dispatch('usersModule/getUsers')
      })
    },
    editUser (e, id) {
      this.editUserInfo.id = id
      this.$store.dispatch('usersModule/getUser', {userId: this.editUserInfo.id}).then(() => {
        this.editableCategories()
        this.editableBrands()
        this.editUserInfo.dialog = true
      })
    },
    editableCategories () {
      this.editUserInfo.categories.model = this.getUser.categories.map(e => String(e.id))
      this.editUserInfo.categories.names = this.getUser.categories.map(e => e.name)
      this.editUserInfo.categories.isLoading = true
      this.$store.dispatch('usersModule/getNewUserCategories', {
        categoryName: 'a',
        pageId: 'users'
      }).then(() => {
        this.editUserInfo.categories.isLoading = false
      }).catch(() => {
        this.editUserInfo.categories.isLoading = false
      })
    },
    editableBrands () {
      this.editUserInfo.brands.model = this.getUser.brands.map(e => String(e.id))
      this.editUserInfo.brands.names = this.getUser.brands.map(e => e.name)
      this.editUserInfo.brands.isLoading = true
      this.$store.dispatch('usersModule/getNewUserBrands', {
        brandName: 'a',
        pageId: 'users'
      }).then(() => {
        this.editUserInfo.brands.isLoading = false
      }).catch(() => {
        this.editUserInfo.brands.isLoading = false
      })
    },
    updateBusinessTitle (e) {
      this.$store.commit('usersModule/updateBusinessTitle', e)
      this.editUserInfo.businessName = e
    },
    updateFullName (e) {
      this.$store.commit('usersModule/updateFullName', e)
    },
    updateUsername (e) {
      this.$store.commit('usersModule/updateUsername', e)
    }
  },
  mounted () {
    this.$store.dispatch('usersModule/getUsers')
  },
  computed: {
    getLoader () {
      return this.$store.state.usersModule.loading
    },
    getNoData () {
      return this.$store.state.usersModule.noData
    },
    getUsers () {
      return this.$store.state.usersModule.users
    },
    getNewUserCategories () {
      return this.$store.state.usersModule.newUser.categories
    },
    getNewUserBrands () {
      return this.$store.state.usersModule.newUser.brands
    },
    getUser () {
      return this.$store.state.usersModule.editUser
    },
    getThemeColor () {
      return this.$store.state.auth.userInfo.themeColor
    }
  },
  watch: {
    'addUser.categories.search': function (val) {
      if (val === '' || !val) {
        this.addUser.categories.isLoading = false
        return
      }
      if (this.timer) {
        clearTimeout(this.timer)
        this.timer = null
      }
      this.timer = setTimeout(() => {
        this.addUser.categories.isLoading = true
        this.$store.dispatch('usersModule/getNewUserCategories', {
          categoryName: val,
          pageId: 'users'
        }).then(() => {
          this.addUser.categories.isLoading = false
        }).catch(() => {
          this.addUser.categories.isLoading = false
        })
      }, 300)
    },
    'addUser.brands.search': function (val) {
      if (val === '' || !val) {
        this.addUser.brands.isLoading = false
        return
      }
      if (this.timer) {
        clearTimeout(this.timer)
        this.timer = null
      }
      this.timer = setTimeout(() => {
        this.addUser.brands.isLoading = true
        this.$store.dispatch('usersModule/getNewUserBrands', {
          brandName: val,
          pageId: 'users'
        }).then(() => {
          this.addUser.brands.isLoading = false
        }).catch(() => {
          this.addUser.brands.isLoading = false
        })
      }, 300)
    },
    'editUserInfo.categories.search': function (val) {
      console.log('val', val)
      if (val === '' || !val) {
        this.editUserInfo.categories.isLoading = false
        return
      }
      if (this.timer) {
        clearTimeout(this.timer)
        this.timer = null
      }
      this.timer = setTimeout(() => {
        this.editUserInfo.categories.isLoading = true
        this.$store.dispatch('usersModule/getNewUserCategories', {
          categoryName: val,
          pageId: 'users'
        }).then(() => {
          this.editUserInfo.categories.isLoading = false
        }).catch(() => {
          this.editUserInfo.categories.isLoading = false
        })
      }, 300)
    }
  }
}
</script>

<style scoped>

</style>
