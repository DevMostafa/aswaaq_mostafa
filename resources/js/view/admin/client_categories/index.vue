<template>
    <div  :class="['page-wrapper',this.$i18n.locale == 'ar'? 'page-wrapper-ar':'']">
        <div class="content container-fluid">

            <!-- Page Header -->
            <div class="page-header">
                <div class="row align-items-center">
                    <div class="col">
                        <h3 class="page-title">{{$t('global.Client Categories')}}</h3>
                        <ul class="breadcrumb">
                            <li class="breadcrumb-item">
                                <router-link :to="{name: 'dashboard'}">
                                    {{$t('dashboard.Dashboard')}}
                                </router-link>
                            </li>
                            <li class="breadcrumb-item active">{{$t('global.Client Categories')}}</li>
                        </ul>
                    </div>

                </div>
            </div>
            <!-- /Page Header -->
            <!-- Table -->
            <div class="row">
                <div class="col-lg-12">
                    <div class="card">
                        <loader v-if="loading"/>
                        <div class="card-body">
                            <div class="card-header pt-0">
                                <div class="row justify-content-between">
                                    <div class="col-5">
                                        {{$t('global.Search')}} :
                                        <input type="search" v-model="search" class="custom"/>
                                    </div>
                                    <div class="col-5 row justify-content-end">
                                        <router-link
                                            v-if="permission.includes('client category create')"
                                           :to="{name: 'createClientCategories'}"
                                            class="btn btn-custom btn-warning">
                                            {{$t('global.Add')}}
                                        </router-link>
                                    </div>
                                </div>
                            </div>
                            <div class="table-responsive">
                                <table class="table mb-0">
                                    <thead>
                                    <tr>
                                        <th>#</th>
                                        <th>{{$t('global.From Amount')}}</th>
                                        <th>{{$t('global.To Amount')}}</th>
                                        <th>{{$t('global.Actions')}}</th>
                                    </tr>
                                    </thead>
                                    <tbody v-if="ClientCategories.length">
                                    <tr v-for="(item,index) in ClientCategories"  :key="item.id">
                                        <td>{{ index + 1 }}</td>
                                        <td>{{ item.from_amount }}</td>
                                        <td>{{ item.to_amount }}</td>

                                        <td>

                                            <router-link
                                                :to="{name: 'editClientCategories',params:{id:item.id}}"
                                               v-if="permission.includes('client category edit')"
                                               class="btn btn-sm btn-success me-2">
                                                <i class="far fa-edit"></i>
                                            </router-link>
                                            <a href="#" @click="deleteClientCategories(item.id,index)"
                                                v-if="permission.includes('client category delete')"
                                               data-bs-target="#staticBackdrop" class="btn btn-sm btn-danger me-2">
                                                <i class="far fa-trash-alt"></i>
                                            </a>
                                        </td>

                                    </tr>
                                    </tbody>
                                    <tbody v-else>
                                        <tr>
                                            <th class="text-center" colspan="5">{{$t('global.No Data Found')}}</th>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <!-- /Table -->
            <!-- start Pagination -->
            <Pagination :limit="2" :data="ClientCategoriesPaginate" @pagination-change-page="getClientCategories">
                <template #prev-nav>
                    <span>&lt; {{$t('global.Previous')}}</span>
                </template>
                <template #next-nav>
                    <span>{{$t('global.Next')}} &gt;</span>
                </template>
            </Pagination>
            <!-- end Pagination -->
        </div>
        <!-- /Page Wrapper -->
    </div>
</template>

<script>
import {onMounted, watch, ref,computed} from "vue";
import {useStore} from "vuex";
import adminApi from "../../../api/adminAxios";
import { useI18n } from "vue-i18n";
export default {
    name: "index",
    setup() {

        // get packages
        let {t} = useI18n();
        let ClientCategories = ref([]);
        let ClientCategoriesPaginate = ref({});
        let loading = ref(false);
        const search = ref('');
        let store = useStore();

        let permission = computed(() => store.getters['authAdmin/permission']);

        let getClientCategories = (page = 1) => {
            loading.value = true;

            adminApi.get(`/v1/dashboard/client_categories?page=${page}&search=${search.value}`)
                .then((res) => {
                    let l = res.data.data;
                    ClientCategoriesPaginate.value = l.client_categories;
                    ClientCategories.value = l.client_categories.data;
                })
                .catch((err) => {
                    console.log(err.response.data);
                })
                .finally(() => {
                    loading.value = false;
                });
        }

        onMounted(() => {
            getClientCategories();
        });

        watch(search, (search, prevSearch) => {
            if (search.length >= 0) {
                getClientCategories();
            }
        });

        let dateFormat = (item) => {
            let now = new Date(item);
            let st = `
                 ${now.getUTCHours()}:${now.getUTCMinutes()}:${now.getUTCSeconds()}
                ${now.getUTCFullYear().toString()}
                 /${(now.getUTCMonth() + 1).toString()}
                 /${now.getUTCDate()}
            `;
            return st;
        };


        function deleteClientCategories(id, index) {
            Swal.fire({
                title: t('global.AreYouSureDelete'),
                text: t('global.YouWontBeAbleToRevertThis'),
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                cancelButtonText: t('global.No'),
confirmButtonText: t('global.Yes')
            }).then((result) => {
                if (result.isConfirmed) {
                    adminApi.delete(`/v1/dashboard/client_categories/${id}`)
                        .then((res) => {
                            ClientCategories.value.splice(index,  1);
                            Swal.fire({
                                icon: 'success',
                                title:t('global.DeletedSuccessfully'),
                                showConfirmButton: false,
                                timer: 1500
                            });
                        }).catch((err) => {
                            Swal.fire({
                                icon: 'error',
                                title: t('global.There is an error'),
                                text: t('global.ThereIsAnErrorInTheSystem'),
                            });
                        });
                }
            });
        }


        return {dateFormat,ClientCategories, loading,permission, getClientCategories, search, deleteClientCategories, ClientCategoriesPaginate};

    },
    data() {
        return {
            locale: localStorage.getItem("langAdmin")
        }
    }
}
</script>

<style scoped>
.card {
    position: relative;
}



.custom {
    border: 1px solid #D7D7D7;
    padding: 2px;
    border-radius: 5px;
    width: 35%;
}

.btn {
    color: #fff;
}
.hover:hover{
    border: 2px solid;
    padding: 3px;
    border-radius: 7px;
}

</style>
