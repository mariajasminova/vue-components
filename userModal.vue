<template>
    <!--Модальное окно для добавления/редактирования пользователя -->
    <b-modal v-model="modal"
             id="user-modal">
        <div class="error-block">
            <template v-for="error in errors">
                <p>{{ error.message }}</p>
            </template>
        </div>
        <!--Header модального окна-->
        <template v-slot:modal-header>
            <h5 class="modal-title w-100 text-center">{{ titleModal }}</h5>
            <button class="close" type="button" data-dismiss="modal" @click="hideModal">
                <span>&times;</span>
            </button>
        </template>
        <!--Body модального окна-->
        <b-form class="w-100">
            <b-form-group id="first-name-group"
                          label="First name"
                          label-for="first-name-input">
                <b-form-input id="first-name-input"
                              v-model="userItem.first_name"
                              type="text"
                              required>
                </b-form-input>
            </b-form-group>
            <b-form-group id="last-name-group"
                          label="Last name"
                          label-for="last-name-input">
                <b-form-input id="last-name-input"
                              v-model="userItem.last_name"
                              type="text"
                              required>
                </b-form-input>
            </b-form-group>
            <b-form-group id="email-group"
                          label="e-mail"
                          label-for="email-input">
                <b-form-input id="email-input"
                              v-model="userItem.email"
                              type="email"
                              required>
                </b-form-input>
            </b-form-group>
            <b-form-group id="role-group"
                          label="Role"
                          label-for="role-select">
                <b-form-select id="role-select"
                               v-model="userItem.role"
                               :options="optionsSelectRole"
                               required>
                </b-form-select>
            </b-form-group>
            <div class="custom-control custom-checkbox form-group modal-checkbox">
                <input id="status-checkbox"
                       type="checkbox"
                       v-model="userItem.status"
                       class="custom-control-input">
                <label class="custom-control-label"
                       for="status-checkbox"
                >Status</label>
            </div>
        </b-form>
        <!--Footer модального окна-->
        <template v-slot:modal-footer>
            <b-button variant="primary"
                      class="float-right "
                      size="sm"
                      @click="submit(userItem)"
            >Save
            </b-button>
            <b-button variant="outline-primary"
                      class="float-right "
                      size="sm"
                      @click="hideModal"
            >Cancel
            </b-button>
        </template>
    </b-modal>
</template>

<script>
    export default {
        name: "userModal",
        props: {
            userItem: {
                type: Object,
                default: null,
                required: true
            },
            optionsSelectRole: {
                type: Array,
                default: null,
                required: true
            },
            titleModal: {
                type: String,
                default: null,
                required: true
            }
        },
        data: function () {
            return {
                errors: null,
                modal: false,
                api: {
                    createUser: '/api/users/create',
                    updateUser: '/api/users/update'
                }
            }
        },
        watch: {
            modal: function () {
                if (this.modal === true) {
                    return;
                }

                this.$emit('hideModal');
            }
        },
        methods: {
            getUrl: function () {
                let url = this.api.create;

                if (null !== this.userItem.id) {
                    url = this.api.update + '/' + this.userItem.id
                }

                return url;
            },
            submit: function (item) {
                this.loading = true;
                this.errors = null;

                const form = new FormData;

                form.append('first_name', item.first_name);
                form.append('last_name', item.last_name);
                form.append('email', item.email);
                form.append('role', item.role);
                form.append('status', item.status);

                this.axios.post(this.getUrl(), form).then(response => {
                    this.modal = false;
                    this.$emit('createItem');
                }).catch((error) => {
                    this.errors = error.response.data;
                    if (422 === error.response.status) {
                        alert(error.response.data.map(e => e.message).join("\n"));
                    } else {
                        alert('Something went wrong, please try again.');
                    }
                }).finally(_ => {
                    this.loading = false;
                });
            },
            showModal() {
                this.modal = true;
            },
            hideModal() {
                this.modal = false;
            },
        }
    }
</script>

<style scoped>

    .modal-checkbox {
        margin: 0;
    }

    .error-block {
        color: red;
    }

</style>