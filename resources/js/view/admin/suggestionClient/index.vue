<template>
    <div  :class="['page-wrapper',this.$i18n.locale == 'ar'? 'page-wrapper-ar':'']">
        <div class="content container-fluid">

            <!-- Page Header -->
            <div class="page-header">
                <div class="row align-items-center">
                    <div class="col">
                        <h3 class="page-title">{{ $t('global.SuggestionClients') }}</h3>
                        <ul class="breadcrumb">
                            <li class="breadcrumb-item">
                                <router-link :to="{name: 'dashboard'}">
                                    {{ $t('dashboard.Dashboard') }}
                                </router-link>
                            </li>
                            <li class="breadcrumb-item active">{{ $t('global.SuggestionClients') }}</li>
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
                                    <div class="col-7">
                                        {{ $t('global.Search') }}:
                                        <input type="search" v-model="search" class="custom"/>
                                    </div>

                                </div>
                            </div>
                            <div class="table-responsive">
                                <table class="table mb-0">
                                    <thead>
                                    <tr>
                                        <th>#</th>
                                        <th>{{ $t('global.Suggestions') }}</th>
                                        <th>{{ $t('global.ClientName') }}</th>
                                        <th>{{ $t('global.complaint') }}</th>
                                        <th>{{ $t('global.Date') }}</th>
                                        <th>{{ $t('global.Action') }}</th>
                                    </tr>
                                    </thead>
                                    <tbody v-if="jobs.length">
                                    <tr v-for="(item,index) in jobs" :key="item.id">
                                        <td>{{ index + 1 }}</td>
                                        <td>{{ item.suggestion.name }}</td>
                                        <td>{{ item.user.name }}</td>
                                        <td>{{ item.notes }}</td>
                                        <th>{{dateFormat(item.created_at)}}</th>
                                        <td>

                                            <router-link
                                                :to="{name: 'showSuggestionClient',params:{id:item.id}}"
                                                v-if="permission.includes('SuggestionClients read')"
                                                class="btn btn-sm btn-info me-2">
                                                <i class="fas fa-book-open"></i> {{$t('global.Show')}}
                                            </router-link>
                                        </td>

                                    </tr>
                                    </tbody>
                                    <tbody v-else>
                                        <tr>
                                             <th class="text-center" colspan="7">{{ $t('global.NoDataFound') }}</th>
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
            <Pagination :limit="2" :data="jobsPaginate" @pagination-change-page="getJob">
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
import {useI18n} from 'vue-i18n';

export default {
    name: "index",
    setup() {
        const {t} = useI18n({});

        let jobs = ref([]);
        let jobsPaginate = ref({});
        let loading = ref(false);
        const search = ref('');
        let store = useStore();

        let permission = computed(() => store.getters['authAdmin/permission']);

        let getJob = (page = 1) => {
            loading.value = true;

            adminApi.get(`/v1/dashboard/suggestionClient?page=${page}&search=${search.value}`)
                .then((res) => {
                    let l = res.data.data;
                    jobsPaginate.value = l.suggestions;
                    jobs.value = l.suggestions.data;
                })
                .catch((err) => {
                    console.log(err.response.data);
                })
                .finally(() => {
                    loading.value = false;
                });
        }

        onMounted(() => {
            getJob();
        });

        watch(search, (search, prevSearch) => {
            if (search.length >= 0) {
                getJob();
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

        return {jobs, loading, getJob,dateFormat, search,permission, jobsPaginate};

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
