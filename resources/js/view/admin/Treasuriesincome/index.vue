<template>
    <div :class="['page-wrapper', this.$i18n.locale == 'ar' ? 'page-wrapper-ar' : '']">
        <div class="content container-fluid">

            <!-- Page Header -->
            <div class="page-header">
                <div class="row align-items-center">
                    <div class="col">
                        <h3 class="page-title">{{ $t('global.treasuriesIncome') }}</h3>
                        <ul class="breadcrumb">
                            <li class="breadcrumb-item">
                                <router-link :to="{ name: 'dashboard', params: { lang: locale || 'ar' } }">
                                    {{ $t('dashboard.Dashboard') }}
                                </router-link>
                            </li>
                            <li class="breadcrumb-item active">{{ $t('global.treasuriesIncome') }}</li>
                        </ul>
                    </div>

                </div>
            </div>
            <!-- /Page Header -->
            <div class="card bg-white projects-card">
                <div class="card-body pt-0">
                    <div class="card-header pt-0">
                        <div class="row justify-content-between">
                            <div class="col-4 row ">
                                <select v-model="treasury_id" class="form-select">
                                    <option value="0">{{ $t('treasury.ChooseTreasury') }}</option>
                                    <option v-for="treasury in treasuries" :key="treasury.id" :value="treasury.id">
                                        {{ treasury.name }}</option>
                                </select>

                            </div>
                            <div v-if="treasury_id" class="col-12 row mt-3">
                                <div class="col-4">
                                    <div class="amount">
                                        <h5>
                                            {{ $t('global.NetTreasury') }} : {{ amount }}
                                        </h5>
                                    </div>
                                </div>
                                <div class="col-4">
                                    <div class="amount">
                                        <h5>
                                            {{ $t('global.TotalIncome') }} : {{ income }}
                                        </h5>
                                    </div>
                                </div>
                                <div class="col-4">
                                    <div class="amount">
                                        <h5>
                                            {{ $t('global.TotalExpense') }} : {{ expense }}
                                        </h5>
                                    </div>
                                </div>
                                <div class="col-4">
                                    <div class="amount">
                                        <h5>
                                            {{ $t('global.incomeTransfer') }} : {{ income_transfer }}
                                        </h5>
                                    </div>
                                </div>
                                <div class="col-4">
                                    <div class="amount">
                                        <h5>
                                            {{ $t('global.expenseTransfer') }} : {{ expense_transfer }}
                                        </h5>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="reviews-menu-links">
                        <ul role="tablist" class="nav nav-pills card-header-pills nav-justified">
                            <li class="nav-item">
                                <a href="#tab-4" data-bs-toggle="tab" class="nav-link active">{{ $t('global.IncomeItems')
                                }}</a>
                            </li>
                            <!-- <li class="nav-item">
                                <a href="#tab-5" data-bs-toggle="tab" class="nav-link">{{ $t('global.clientsIncome') }}</a>
                            </li>
                            <li class="nav-item">
                                <a href="#tab-6" data-bs-toggle="tab" class="nav-link">{{ $t('global.SuppliersIncome') }}</a>
                            </li> -->
                            <!-- <li class="nav-item">
                                <a href="#tab-7" data-bs-toggle="tab" class="nav-link">{{ $t('global.capitalOwnerAccount') }}</a>
                            </li> -->
                        </ul>
                    </div>
                    <div class="tab-content pt-0">
                        <div role="tabpanel" id="tab-4" class="tab-pane fade active show">
                            <loader v-if="loading" />
                            <div class="row justify-content-between">
                                <div class="col-5">
                                    {{ $t('global.Search') }}:
                                    <input type="search" v-model="search" class="custom" />
                                </div>
                            </div>
                            <div class="table-responsive">
                                <table class="table table-center table-hover mb-0 datatable">
                                    <thead>
                                        <tr>
                                            <th class="text-center">#</th>
                                            <th class="text-center">{{ $t('global.Band Name') }}</th>
                                            <th class="text-center">{{ $t('global.Amount') }}</th>
                                            <th class="text-center">{{ $t('global.NameOfThePayer') }}</th>
                                            <th class="text-center">{{ $t('global.For') }}</th>
                                            <th class="text-center">{{ $t('global.PaymentDate') }}</th>
                                            <th class="text-center">{{ $t('global.TransactionDate') }}</th>
                                            <th class="text-center">{{ $t('global.ProcessWriter') }}</th>
                                        </tr>
                                    </thead>
                                    <tbody v-if="incomes.length">
                                        <tr v-for="(item) in incomes" :key="item.id">
                                            <td class="text-center">{{ item.id }}</td>
                                            <td class="text-center">{{ item.income.name }}</td>
                                            <td class="text-center">{{ item.amount }}</td>
                                            <td class="text-center">{{ item.payer }}</td>
                                            <td class="text-center">{{ item.notes }}</td>
                                            <td class="text-center">{{ item.payment_date }}</td>
                                            <td class="text-center">{{ dateFormat(item.created_at) }}</td>
                                            <td class="text-center">{{ item.user ? item.user.name : "-----" }}</td>

                                        </tr>
                                    </tbody>
                                    <tbody v-else>
                                        <tr>
                                            <th class="text-center" colspan="8">{{ $t('global.NoDataFound') }}</th>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                            <!-- start Pagination -->
                            <Pagination :limit="2" :data="incomesPaginate" @pagination-change-page="getIncome">
                                <template #prev-nav>
                                    <span>&lt; {{ $t('global.Previous') }}</span>
                                </template>
                                <template #next-nav>
                                    <span>{{ $t('global.Next') }} &gt;</span>
                                </template>
                            </Pagination>
                            <!-- end Pagination -->
                        </div>

                    </div>
                    </div>
                </div>
            </div>
            <!-- /Page Wrapper -->
        </div>
</template>

<script>
import { onMounted, inject, watch, ref } from "vue";
import { useStore } from "vuex";
import adminApi from "../../../api/adminAxios";
import { useI18n } from "vue-i18n";

export default {
    name: "index",
    setup() {
        const emitter = inject('emitter');
        const { t } = useI18n({});

        // get packages
        let incomes = ref([]);
        let treasuries = ref([]);
        let incomesPaginate = ref({});
        let treasury_id = ref(0);
        let amount = ref(0);
        let income = ref(0);
        let expense = ref(0);
        let income_transfer = ref(0);
        let expense_transfer = ref(0);
        let loading = ref(false);
        const search = ref('');

        let getIncome = (page = 1) => {
            if (treasury_id.value > 0) {
                loading.value = true;

                adminApi.get(`/v1/dashboard/treasuriesIncome/${treasury_id.value}?page=${page}&search=${search.value}`)
                    .then((res) => {
                        let l = res.data.data;
                        incomesPaginate.value = l.incomes;
                        incomes.value = l.incomes.data;
                    })
                    .catch((err) => {
                        console.log(err.response.data);
                    })
                    .finally(() => {
                        loading.value = false;
                    });
            }
        }


        let getTreasuries = () => {
            loading.value = true;

            adminApi.get(`/v1/dashboard/activeTreasury`)
                .then((res) => {
                    let l = res.data.data;
                    treasuries.value = l.treasuries;
                })
                .catch((err) => {
                    console.log(err.response.data);
                })
                .finally(() => {
                    loading.value = false;
                })
        }

        onMounted(() => {
            getIncome();
            getTreasuries();
        });

        watch(search, (search, prevSearch) => {
            if (search.length >= 0) {
                getIncome();
            }
        });



        watch(treasury_id, (treasury_id, prevSearch) => {
            let v = treasuries.value.find((el) => el.id == treasury_id)

            amount.value = v.amount;
            income.value = v.income;
            expense.value = v.expense;
            income_transfer.value = v.income_transfer;
            expense_transfer.value = v.expense_transfer;
            getIncome();
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
        }

        return { expense, income, amount, income_transfer, expense_transfer, treasury_id, incomes, treasuries, loading, getIncome, dateFormat, search, incomesPaginate };

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

.hover:hover {
    border: 2px solid;
    padding: 3px;
    border-radius: 7px;
}


.amount {
    background-color: #fcb00c;
    color: #000;
    padding: 10px;
    margin-top: 3px;
}

.amount h5 {
    text-align: center;
    color: #fff;
}
</style>
