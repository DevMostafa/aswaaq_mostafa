<template>
    <div  :class="['page-wrapper',this.$i18n.locale == 'ar'? 'page-wrapper-ar':'']">
        <div class="content container-fluid">

            <!-- Page Header -->
            <div class="page-header">
                <div class="row align-items-center">
                    <div class="col">
                        <h3 class="page-title">{{ $t('suppiler.suppiler') }}</h3>
                        <ul class="breadcrumb">
                            <li class="breadcrumb-item">
                                <router-link :to="{name: 'dashboard'}">
                                    {{ $t('dashboard.Dashboard') }}
                                </router-link>
                            </li>
                            <li class="breadcrumb-item active">{{ $t('suppiler.suppiler') }}</li>
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
                                        {{ $t('global.Search') }}:
                                        <input type="search" v-model="search" class="custom"/>
                                    </div>
                                    <div class="col-5 row justify-content-end">
                                        <router-link
                                            v-if="permission.includes('supplier create')"
                                           :to="{name: 'createSupplier'}"
                                            class="btn btn-custom btn-warning">
                                            {{ $t('global.Add') }}
                                        </router-link>
                                    </div>
                                </div>
                            </div>
                            <div class="table-responsive">
                                <table class="table mb-0">
                                    <thead>
                                    <tr>
                                        <th>#</th>
                                        <th>{{ $t('global.Name') }}</th>
                                        <th>{{ $t('global.Phone') }}</th>
                                        <th>{{ $t("global.Address") }}</th>
                                        <th>{{ $t("global.Total Commission") }}</th>
                                        <th>{{ $t("global.Number of Orders") }}</th>
                                        <th>{{ $t("global.IsAswaaqSupplier") }}</th>
                                        <th>{{ $t('global.Status') }}</th>
                                        <th>{{ $t('global.Action') }}</th>
                                    </tr>
                                    </thead>
                                    <tbody v-if="suppliers.length">
                                    <tr v-for="(item,index) in suppliers"  :key="item.id">
                                        <td>{{ item.id }}</td>
                                        <td>{{ item.name }}</td>
                                        <td>{{ item.phone }}</td>
                                        <td>{{ item.address }}</td>
                                        <td>{{ item.total_commission }}</td>
                                        <td>{{ item.order_numbers }}</td>
                                        <td class="text-center">
                                            <span class="badge badge-primary" v-if="item.is_our_supplier">
                                            {{ $t("global.IsAswaaqSupplier") }}
                                            </span>
                                        </td>
                                        <td>
                                            <a href="#" @click="activationSupplier(item.id,item.status,index)">
                                                <span :class="[parseInt(item.status) ? 'text-success hover': 'text-danger hover']">{{
                                                        parseInt(item.status) ? $t('treasury.Active') : $t('treasury.Inactive') }}</span>
                                            </a>
                                        </td>
                                        <td>

                                            <router-link :to="{name:'supplier_profile',params:{id:item.id,lang:this.$i18n.locale}}"
                                                data-toggle="modal"
                                                class="btn btn-sm btn-info me-2"
                                                >
                                                <i class="far fa-eye"></i>
                                            </router-link>
                                            <router-link
                                                :to="{name: 'editSupplier',params:{id:item.id}}"
                                               v-if="permission.includes('supplier edit')"
                                               class="btn btn-sm btn-success me-2">
                                                <i class="far fa-edit"></i>
                                            </router-link>
                                            <router-link :to="{
                                                    name: 'showVirtualStock',
                                                    params: { id: item.id },
                                                }" v-if="permission.includes('virtualStock read')" class="btn btn-sm btn-info me-2">
                                                    <i class="fas fa-book-open"></i>
                                                    {{ $t("global.ShowDetails") }}
                                                </router-link>

                                                <router-link :to="{
                                                    name: 'createVirtualStock',
                                                    params: { id: item.id },
                                                }" v-if="permission.includes('virtualStock create')" class="btn btn-sm btn-warning me-2">
                                                    <i class="fas fa-truck"></i>
                                                    {{ $t("global.Upload Prices") }}
                                                </router-link>
                                            <!-- <a href="#" @click="deleteSupplier(item.id,index)"
                                                v-if="permission.includes('supplier delete')"
                                               data-bs-target="#staticBackdrop" class="btn btn-sm btn-danger me-2">
                                                <i class="far fa-trash-alt"></i>
                                            </a> -->
                                        </td>

                                    </tr>
                                    </tbody>
                                    <tbody v-else>
                                        <tr>
                                            <th class="text-center" colspan="5">{{ $t('global.NoDataFound') }}</th>
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
            <Pagination
                :data="suppliersPaginate"
                @pagination-change-page="getSupplier"
                :limit="2"
            >
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
import {onMounted, inject, watch, ref,computed} from "vue";
import {useStore} from "vuex";
import adminApi from "../../../api/adminAxios";
import {useI18n} from "vue-i18n";

export default {
    name: "index",

    setup() {
        // get packages
        let suppliers = ref([]);
        let suppliersPaginate = ref({});
        let loading = ref(false);
        const search = ref('');
        let store = useStore();
        const {t} = useI18n({});

        let permission = computed(() => store.getters['authAdmin/permission']);

        let getSupplier = (page = 1) => {
            loading.value = true;

            adminApi.get(`/v1/dashboard/supplier?page=${page}&search=${search.value}`)
                .then((res) => {
                    let l = res.data.data;
                    suppliersPaginate.value = l.suppliers;
                    suppliers.value = l.suppliers.data;
                })
                .catch((err) => {
                    console.log(err.response.data);
                })
                .finally(() => {
                    loading.value = false;
                });
        }

        onMounted(() => {
            getSupplier();
        });

        watch(search, (search, prevSearch) => {
            if (search.length >= 0) {
                getSupplier();
            }
        });


        function deleteSupplier(id, index) {
            Swal.fire({
                title: `${t('global.AreYouSureDelete')} `,
                text: `${t("global.YouWontBeAbleToRevertThis")}`,
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                cancelButtonText: t('global.No'),
confirmButtonText: t('global.Yes'),
            }).then((result) => {
                if (result.isConfirmed) {

                    adminApi.delete(`/v1/dashboard/supplier/${id}`)
                        .then((res) => {
                            suppliers.value.splice(index,  1);

                            Swal.fire({
                                icon: 'success',
                                title: `${t("global.DeletedSuccessfully")}`,
                                showConfirmButton: false,
                                timer: 1500
                            });
                        })
                        .catch((err) => {
                            Swal.fire({
                                icon: 'error',
                                title: `${t('global.ThereIsAnErrorInTheSystem')}`,
                                text: `${t('global.YouCanNotDelete')}`,
                            });
                        });
                }
            });
        }

        function activationSupplier(id, active,index) {
            Swal.fire({
                title: `${parseInt(active) ? t('global.AreYouSureInactive') :t('global.AreYouSureActive')}`,
                text: `${t("global.YouWontBeAbleToRevertThis")}`,
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                cancelButtonText: t('global.No'),
confirmButtonText: t('global.Yes')
            }).then((result) => {
                if (result.isConfirmed) {

                    adminApi.get(`/v1/dashboard/activationSupplier/${id}`)
                        .then((res) => {
                            Swal.fire({
                                icon: 'success',
                                title: `${parseInt(active)  ? t('global.InactiveSuccessfully') :t('global.ActiveSuccessfully')}`,
                                showConfirmButton: false,
                                timer: 1500
                            });
                            suppliers.value[index]['status'] =  parseInt(active)  ? 0:1
                        })
                        .catch((err) => {
                            Swal.fire({
                                icon: 'error',
                                title: `${t('global.ThereIsAnErrorInTheSystem')}`,
                                text: `${t('global.YouCanNotDelete')}`,
                            });
                        });
                }
            });
        }

        return {suppliers, loading,permission, getSupplier, search, deleteSupplier, activationSupplier, suppliersPaginate};

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
