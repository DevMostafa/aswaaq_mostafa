<template>
    <div class="supplier-container" :class="[
        'page-wrapper',
        this.$i18n.locale == 'ar' ? 'page-wrapper-ar' : '',
    ]">
        <div class="content container-fluid">
            <!-- Page Header -->
            <div class="page-header">
                <div class="row align-items-center">
                    <div class="col">
                        <h3 class="page-title">{{ $t("global.Supplier Profile") }}</h3>
                        <ul class="breadcrumb">
                            <li class="breadcrumb-item">
                                <router-link :to="{ name: 'dashboard' }">
                                    {{ $t("dashboard.Dashboard") }}
                                </router-link>
                            </li>
                            <li class="breadcrumb-item active">
                                {{ $t("global.Supplier Profile") }}
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <!-- /Page Header -->
            <!-- Table -->
            <div class="row">
                <div class="col-lg-8 mb-5">
                    <div class="card">
                        <loader v-if="loading" />
                        <div class="card-body">
                            <div class="card-header pt-0">
                                <div class="row justify-content-between">
                                    <div class="col-md-12 col-sm-12">
                                        {{ $t("global.Products") }}
                                    </div>

                                    <div class="form-group col-md-4 col-sm-12">
                                        <input id="search" type="text" class="form-control"
                                            :placeholder="$t('global.Search')" v-model="search_products" />
                                    </div>
                                    <div class="form-group col-md-4 col-sm-12">
                                        <select v-model="filter" class="form-control" id="Filter"
                                            @change="getSupplierProducts()">
                                            <option value="">{{ $t('global.All Products') }}</option>
                                            <option value="active">{{ $t('global.Active Products') }}</option>
                                            <option value="inactive">{{ $t('global.Inactive Products') }}</option>
                                        </select>
                                    </div>
                                    <div class="form-group col-md-4 col-sm-12">
                                        <router-link :to="{ name: 'indexSupplier' }" class="btn btn-custom btn-dark w-50">
                                            {{ $t('global.back') }}
                                        </router-link>
                                        <a class="btn btn-sm btn-secondary mx-2"
                                            @click.prevent="printSection(supplier.name, '#supplierProductsDetails')"><i
                                                class="fa fa-print"></i> </a>
                                    </div>
                                </div>
                            </div>
                            <div class="table-responsive">
                                <table class="table mb-0" id="supplierProductsDetails">
                                    <thead>
                                        <tr>
                                            <th>{{ $t("global.Action") }}</th>
                                            <th class="text-center">{{ $t("global.Product") }}</th>
                                            <th class="text-center">{{ $t("global.Flavor") }}</th>
                                            <th class="text-center">{{ $t("global.Size") }}</th>
                                            <th class="text-center">{{ $t("global.Main Measurement Unit") }}</th>
                                            <th class="text-center">{{ $t("global.Sub Measurement Unit") }}</th>
                                            <th class="text-center">{{ $t("global.Main Measurement Unit Quantity") }}</th>
                                            <th class="text-center">{{ $t("global.Sub Measurement Unit Quantity") }}</th>
                                            <th class="text-center">{{ $t("global.Main Measurement Unit Price") }}</th>
                                            <th class="text-center">{{ $t("global.Sub Measurement Unit Price") }}</th>
                                            <th class="text-center">{{ $t("global.Main Measurement Unit Price After Sale") }}</th>
                                            <th class="text-center">{{ $t("global.Sub Measurement Unit Price After Sale") }}</th>
                                        </tr>
                                    </thead>
                                    <tbody v-if="Object.keys(products.data ?? []).length">
                                        <tr v-for="(item, index) in products.data" :key="item.id">
                                            <td>
                                                <a href="javascript:void(0);" class="btn btn-sm btn-info me-2"
                                                    data-bs-toggle="modal" @click.prevent="setProductDetails(item.logs);product_count_unit = item.product.count_unit"
                                                    data-bs-target="#showProductDetails">
                                                    <i class="fa fa-eye"></i>
                                                </a>
                                            </td>
                                            <td class="text-center">{{ item.product.name }}</td>
                                            <td class="text-center">{{ item.product.flavor_name}}</td>
                                            <td class="text-center">{{ item.product.size.name }}</td>
                                            <td class="text-center">{{ item.product.main_measure_name }}</td>
                                            <td class="text-center">{{ item.product.sub_measure_name }}</td>
                                            <td class="text-center">{{ Math.ceil(item.quantity / item.product.count_unit) }}</td>
                                            <td class="text-center">{{ item.quantity }}</td>
                                            <td class="text-center">{{ item.main_measurement_price }}</td>
                                            <td class="text-center">{{ item.sub_measurement_price }}</td>
                                            <td class="text-center">{{ item.main_measurement_price_after_sale }}</td>
                                            <td class="text-center">{{ item.sub_measurement_price_after_sale }}</td>

                                        </tr>
                                    </tbody>
                                    <tbody v-else>
                                        <tr>
                                            <th class="text-center" colspan="7">
                                                {{ $t("global.NoDataFound") }}
                                            </th>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                    <!-- /Table -->
                    <!-- start Pagination -->
                    <Pagination :data="products" :limit="5" @pagination-change-page="getSupplierProducts">
                        <template #prev-nav>
                            <span>&lt; {{ $t("global.Previous") }}</span>
                        </template>
                        <template #next-nav>
                            <span>{{ $t("global.Next") }} &gt;</span>
                        </template>
                    </Pagination>
                    <!-- end Pagination -->
                </div>
                <!-- /Table -->

                <div class="col-lg-4 col-sm-12">
                    <div class="text-center card-box">
                        <div class="text-left">
                            <h4 class="header-title mb-4">{{ $t("global.Supplier") }}</h4>
                        </div>
                        <div class="member-card">
                            <div class="avatar-xl member-thumb mb-2 mx-auto d-block star-div">
                                <img :src="
                                    supplier.image ??
                                    'https://ui-avatars.com/api/?name=' +
                                    supplier.name +
                                    '&amp;color=7F9CF5&amp;background=EBF4FF'
                                " :onerror="
    'https://ui-avatars.com/api/?name=' +
    supplier.name +
    '&amp;color=7F9CF5&amp;background=EBF4FF'
" class="rounded-circle img-thumbnail" alt="profile-image" />
                                <i class="fas fa-certificate text-primary small star-icon" title="Featured Agent"></i>
                            </div>

                            <div class="">
                                <h5 class="font-18 mb-1">{{ supplier.name }}</h5>
                            </div>

                            <div class="mt-20">
                                <ul class="list-inline row">
                                    <li class="list-inline-item col-12 mx-0">
                                        <h5>{{ $t("global.Email") }}</h5>
                                        <p>{{ supplier.email }}</p>
                                    </li>
                                    <li class="list-inline-item col-6 mx-0">
                                        <h5>{{ $t("global.Number of Orders") }}</h5>
                                        <p>{{ supplier.order_numbers }}</p>
                                    </li>

                                    <li class="list-inline-item col-6 mx-0">
                                        <h5>{{ $t("global.Phone") }}</h5>
                                        <p>{{ supplier.phone }}</p>
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <!-- end membar card -->
                    </div>
                </div>
            </div>
            <div class="row">

                <div class="col-lg-12" id="supplierSoldProducts">
                    <div class="card">
                        <loader v-if="loading" />
                        <div class="card-body">
                            <div class="card-header pt-0">
                                <div class="row justify-content-between">
                                    <div class="col-md-2 col-sm-12">
                                        {{ $t("global.Sold Products") }}
                                    </div>
                                    <div class="col-md-10 col-sm-12 row justify-content-between">
                                        <div class="form-group col-md-4 col-sm-12">
                                            <label for="search">{{ $t("global.Search") }}</label>
                                            <input id="search" type="text" class="form-control"
                                                :placeholder="$t('global.Search')" v-model="search_items" />
                                        </div>
                                        <div class="form-group col-md-3 col-sm-12">
                                            <label for="from">{{ $t("global.From") }}</label>
                                            <input id="from" type="date" class="form-control" v-model="date_from" />
                                        </div>
                                        <div class="form-group col-md-3 col-sm-12">
                                            <label for="to">{{ $t("global.To") }}</label>
                                            <input id="to" type="date" class="form-control" v-model="date_to" />
                                        </div>
                                        <div class="form-group col-md-2 col-sm-12">
                                            <label class="d-block">.</label>

                                            <a class="btn btn-sm btn-secondary mx -2"
                                                @click.prevent="printSection(supplier.name, '#supplierSoldProducts')"><i
                                                    class="fa fa-print"></i> </a> <button class="btn btn-info"
                                                @click.prevent="getSupplierOrders()">
                                                {{ $t("global.Search") }}
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="table-responsive">
                                <table class="table mb-0">
                                    <thead>
                                        <tr>
                                            <th>{{ $t("global.Product") }}</th>
                                            <th>{{ $t("global.Flavor") }}</th>
                                            <th>{{ $t("global.Size") }}</th>
                                            <th>{{ $t("global.Measurement Unit") }}</th>
                                            <th>{{ $t("global.Measurement Unit Price") }}</th>
                                            <th>{{ $t("global.sold quantity") }}</th>
                                            <th>{{ $t("global.Returned Quantity") }}</th>
                                            <th>{{ $t("global.Total Amount") }}</th>
                                        </tr>
                                    </thead>
                                    <tbody v-if="Object.keys(orders.data ?? []).length">
                                        <tr v-for="(item, index) in orders.data" :key="item.id">
                                            <td>{{ this.$i18n.locale == 'ar' ? item.product_ar : item.product_en }}</td>
                                            <td>{{ this.$i18n.locale == 'ar' ? item.flavor_ar : item.flavor_en }}</td>
                                            <td>{{ this.$i18n.locale == 'ar' ? item.size_ar : item.size_en }}</td>
                                            <td>{{ this.$i18n.locale == 'ar' ? item.measurement_ar : item.measurement_en }}</td>
                                            <td>
                                                {{ item.price }} {{ $t("global.LE") }}
                                            </td>
                                            <td>{{ item.quantity }}</td>
                                            <td>{{ item.returned_quantity }}</td>
                                            <td>{{ item.total_amount }} {{ $t("global.LE") }}</td>
                                        </tr>
                                    </tbody>
                                    <tbody v-else>
                                        <tr>
                                            <th class="text-center" colspan="7">
                                                {{ $t("global.NoDataFound") }}
                                            </th>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                    <!-- /Table -->

                    <!-- start Pagination -->
                    <Pagination :data="orders" @pagination-change-page="getSupplierOrders" :limit="5">
                        <template #prev-nav>
                            <span>&lt; {{ $t("global.Previous") }}</span>
                        </template>
                        <template #next-nav>
                            <span>{{ $t("global.Next") }} &gt;</span>
                        </template>
                    </Pagination>
                    <!-- end Pagination -->
                </div>
            </div>

            <!-- Edit Modal -->
            <div class="modal fade custom-modal" id="showProductDetails">
                <div class="modal-dialog modal-dialog-centered " style="max-width:1300px!important">
                    <div class="modal-content">
                        <!-- Modal Header -->
                        <div class="modal-header d-block text-center">
                            <h4 class="modal-title">
                                {{ $t("global.Product details") }}
                            </h4>
                            <p>( <span class="text-danger"> {{ $t('global.The number of units within the main measurement unit') }} = {{ product_count_unit }}</span>)</p>
                            <button id="close-showProductDetails" type="button" class="close print-button"
                                data-bs-dismiss="modal">
                                <span>&times;</span>
                            </button>
                        </div>

                        <!-- Modal body -->
                        <div class="modal-body row">
                            <div class="card bg-white projects-card">
                                <div class="card-body pt-0 table-responsive">
                                    <table class="table table-center table-hover mb-0 datatable">
                                        <thead>
                                            <tr>
                                                <th>{{ $t("global.Main Measurement Unit Price") }}</th>
                                                <th>{{ $t("global.Sub Measurement Unit Price") }}</th>
                                                <th>{{ $t("global.Main Measurement Unit Price After Sale") }}</th>
                                                <th>{{ $t("global.Sub Measurement Unit Price After Sale") }}</th>

                                                <th>{{ $t("global.diff quantity") }}</th>
                                                <th>{{ $t("global.sold quantity") }}</th>
                                                <th>{{ $t("global.Total Quantity") }}</th>
                                                <th>{{ $t("global.Date") }}</th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr v-for="product in product_logs" :key="product.id">

                                                <td>{{ product.main_measurement_price }} {{ $t('global.LE') }}</td>
                                                <td>{{ product.sub_measurement_price }} {{ $t('global.LE') }}</td>
                                                <td>{{ product.main_measurement_price_after_sale }} {{ $t('global.LE') }}</td>
                                                <td>{{ product.sub_measurement_price_after_sale }} {{ $t('global.LE') }}</td>
                                                <td>{{ product.diff_qty }}</td>
                                                <td>{{ product.sold_quantity }}</td>
                                                <td>{{ product.total_qty??0 }}</td>
                                                <td>{{ dateFormat(product.created_at) }}</td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <!-- /Edit Modal -->
        </div>
    </div>
</template>

<script>
import { computed, onMounted, provide, watch, ref } from "@vue/runtime-core";
import adminApi from "../../../api/adminAxios";
import { useI18n } from "vue-i18n";
export default {
    props: ["id"],
    setup(props) {
        const { t } = useI18n();
        const products = ref({});
        const loading = ref(false);
        const product_logs = ref({});
        const supplier = ref({});
        const orders = ref({});
        const debounce = ref({});
        const search_items = ref("");
        const product_count_unit = ref("");
        const filter = ref("");
        const search_products = ref("");
        const date_from = ref("");
        const date_to = ref("");
        const getSupplier = async (id) => {
            adminApi.get(`/v1/dashboard/supplier_profile/${id}`).then((response) => {
                supplier.value = response.data.supplier;
            });
        };
        const getSupplierOrders = async (page = 1) => {
            loading.value = true
            await adminApi
                .get(
                    `/v1/dashboard/supplier_orders?page=${page}&user_id=${props.id}&search=${search_items.value}&date_from=${date_from.value}&date_to=${date_to.value}`
                )
                .then((response) => {
                    orders.value = response.data.items;
                });
            loading.value = false

        };
        const getSupplierProducts = async (page = 1) => {
            loading.value = true
            await adminApi
                .get(
                    `/v1/dashboard/supplier_products?page=${page}&user_id=${props.id}&search=${search_products.value}&filter=${filter.value}`
                )
                .then((response) => {
                    products.value = response.data.products;
                });
            loading.value = false

        };

        const printSection = async (supplier_name, id) => {
            $(id).printThis({
                header: `<h1 class="text-center">${supplier_name}</h1>`
            });
        }

        onMounted(() => {
            getSupplier(props.id);
            getSupplierOrders();
            getSupplierProducts();
        });

        watch(search_products, () => {
            clearTimeout(debounce.value);
            debounce.value = setTimeout(() => {
                getSupplierProducts();
            }, 500);
        });

        const setProductDetails = (data) => {
            console.log(data)
            product_logs.value = data;
        };

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

        return {
            product_count_unit,
            supplier,
            dateFormat,
            getSupplierOrders,
            setProductDetails,
            search_items,
            date_from,
            date_to,
            search_products,
            getSupplierProducts,
            products,
            loading,
            orders,
            filter,
            product_logs,
            printSection,
        };
    },
};
</script>

<style lang="scss" scoped>
.card-box {
    background-color: #a1ccff;
    padding: 10px;
    border-radius: 10px;
    margin-bottom: 20px;
}

.supplier-container {
    padding-bottom: 20px;

    .card {
        position: relative;

        .btn-custom {
            width: 30%;
        }

        .custom {
            border: 1px solid #d7d7d7;
            padding: 2px;
            border-radius: 5px;
            width: 45%;
        }

        .btn {
            color: #fff;
        }

        .active {
            background: none;
            border: none;
        }
    }

    .star-div {
        position: relative;
    }

    .star-icon {
        position: absolute;
        top: 4px;
        right: 2px;
        font-size: 16px;
        background-color: #f3f3f3;
        height: 20px;
        width: 20px;
        border-radius: 50%;
        line-height: 20px;
        text-align: center;
    }
}
</style>
