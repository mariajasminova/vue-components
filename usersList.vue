<template>
    <b-container class="px-0">
        <div v-if="errored">
            <div class="error-block">
                <template v-for="error in errors">
                    <p>{{ error.message }}</p>
                </template>
            </div>
            <!--блок описания ошибки, будет заменен на соответствующий компонент -->
        </div>
        <div v-else>
            <div v-if="loading">
                Loading...
            </div> <!--блок описания загрузки, будет заменен на соответствующий компонент -->
            <div v-else>
                <!--кнопка для добавления записи в таблицу-->
                <button type="button"
                        class="button-add btn btn-success btn-sm"
                        @click="addUserForm()"
                >Add
                </button>

                <!--Список пользователей. Таблица-->
                <table class="table table-hover">
                    <thead>
                    <tr>
                        <th>#</th> <!--Порядковый номер в таблице-->
                        <th>ID</th>  <!--id пользователя-->
                        <th>First name</th>
                        <th>Last name</th>
                        <th>e-mail</th>
                        <th>Role</th>
                        <th>Status</th>
                        <th></th>
                    </tr>
                    </thead>
                    <tbody class="td-padding">
                    <tr v-for="(item, index) in items" :key="index">  <!-- Цикл по строкам -->
                        <td class="td-padding">{{ index + 1 }}</td>
                        <td class="td-padding">{{ item.id }}</td>
                        <td>{{ item.first_name }}</td>
                        <td>{{ item.last_name }}</td>
                        <td>{{ item.email }}</td>
                        <td>{{ definitionRoleName(item.role) }}</td>
                        <td class="td-padding">{{ item.status }}</td>
                        <td style="padding: 2px">
                            <!--Кнопки для редактирования и удаления записи (будут заменены на иконки) -->
                            <button type="button"
                                    class="btn btn-warning btn-sm"
                                    @click="editUserForm(item)"
                            >E
                            </button>
                            <button type="button"
                                    class="btn btn-danger btn-sm"
                                    @click="deleteUser(item.id)"
                            >D
                            </button>
                        </td>
                    </tr>
                    </tbody>
                </table>
                <user-modal
                        ref="modalUser"
                        :userItem="userItem"
                        :optionsSelectRole="optionsSelectRole"
                        :titleModal="titleModal"
                        @hideModal="resetUserModal"
                        @createItem="getListUsers">
                </user-modal>
            </div>
        </div>
    </b-container>
</template>

<script>
    import UserModal from "./userModal.vue";

    export default {
        name: "usersList",
        components: {
            UserModal
        },
        data: function () {
            return {
                loading: false, //флаг блока(компонента) загрузки
                errored: false, //флаг блока(компонента) ошибки
                errors: null,
                titleModal: '',
                items: [
                    {
                        id: 1,
                        first_name: "Nik",
                        last_name: "Pet",
                        email: "pet@yan.com",
                        role: 1,
                        status: false
                    }
                ],
                userItem: {
                    id: null,
                    first_name: '',
                    last_name: '',
                    email: '',
                    role: null,
                    status: null
                },
                optionsSelectRole: [
                    {value: null, text: 'Please select an option'},
                    {value: 1, text: 'Super admin'},
                    {value: 2, text: 'Admin'},
                    {value: 3, text: 'User'},
                ],
                api: {
                    searchUsers: '/api/users/search',
                    deleteUser: '/api/users/delete',

                }
            }
        },
        mounted() {
            this.getListUsers();
        },
        methods: {
            getListUsers() {
                this.axios
                    .get(this.api.searchUsers)
                    .then(response => {
                        this.items = response.data.items;
                    }).catch((error) => {
                    this.errors = error.response.data;
                    this.errored = true;
                    if (422 === error.response.status) {
                        alert(error.response.data.map(e => e.message).join("\n"));
                    } else {
                        alert('Something went wrong, please try again.');
                    }
                }).finally(() => {
                    this.loading = false;
                });
            },
            deleteUser(userId) {
                if (false === confirm("Do you really want to delete this user?")) {
                    return;
                }

                let url = this.api.deleteUser + '/' + userId;

                this.axios
                    .post(url)
                    .then(response => {
                        this.getListUsers();
                    }).catch((error) => {
                    this.errors = error.response.data;
                    this.errored = true;
                }).finally(() => {
                    this.loading = false;
                });
            },
            editUserForm(item) {
                this.titleModal = 'Edit User'
                this.userItem = item;
                this.$refs.modalUser.showModal();
            },
            addUserForm() {
                this.titleModal = 'Add User';
                this.$refs.modalUser.showModal();
            },
            resetUserModal() {
                this.userItem = {
                    id: null,
                    first_name: '',
                    last_name: '',
                    email: '',
                    role: null,
                    status: null
                };
            },
            //определение значения роли
            definitionRoleName(role) {
                let text = '';
                switch (role) {
                    case this.optionsSelectRole[1].value :
                        text = this.optionsSelectRole[1].text;
                        break;
                    case this.optionsSelectRole[2].value :
                        text = this.optionsSelectRole[2].text;
                        break;
                    case this.optionsSelectRole[3].value :
                        text = this.optionsSelectRole[3].text;
                        break;
                    default:
                        text = "";
                        break;
                }
                return text;
                //или цикл

            }
        }
    }
</script>

<style scoped>

    .td-padding td {
        padding: 0.4375rem 0 0 0.8125rem;
    }

    .button-add {
        margin-bottom: 1rem;
    }

    .error-block {
        color: red;
    }

</style>