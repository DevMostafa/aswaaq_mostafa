<template>
    <div  :class="['page-wrapper',this.$i18n.locale == 'ar'? 'page-wrapper-ar':'']">

        <div class="content container-fluid">

            <notifications position="top left"  />

            <!-- Page Header -->
            <div class="page-header">
                <div class="row align-items-center">
                    <div class="col">
                        <h3 class="page-title">{{$t('global.Discounts')}}</h3>
                        <ul class="breadcrumb">
                            <li class="breadcrumb-item"><router-link :to="{name: 'indexDiscount'}">{{$t('global.Discount')}}</router-link></li>
                            <li class="breadcrumb-item active">{{$t('global.Add Discount')}}</li>
                        </ul>
                    </div>
                </div>
            </div>
            <!-- /Page Header -->
            <!-- Table -->
            <div class="row">
                <div class="col-lg-12">
                    <div class="card">
                        <loader v-if="loading" />
                        <div class="card-body">
                            <div class="card-header pt-0 mb-4">
                                <router-link
                                    :to="{name: 'indexDiscount'}"
                                    class="btn btn-custom btn-dark"
                                >
                                   {{$t('global.back')}}
                                </router-link>
                            </div>
                            <div class="row">
                                <div class="col-sm">
                                    <form @submit.prevent="storeDiscount" class="needs-validation">
                                        <div class="form-row row">

                                            <div class="col-md-6 mb-3">
                                                <label for="validationCustom01">{{ $t('global.Code') }}</label>
                                                <input
                                                   type="text" class="form-control"
                                                   v-model.trim="v$.code.$model"
                                                   id="validationCustom01"
                                                   :placeholder="$t('global.Code')"
                                                   :class="{'is-invalid':v$.code.$error,'is-valid':!v$.code.$invalid}"
                                                >
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                    <span v-if="v$.code.required.$invalid"> {{ $t('global.IsRequired') }}<br /> </span>
                                                    <span v-if="v$.code.maxLength.$invalid"> {{$t('global.IsMustHaveAtLeast')}} {{ v$.code.minLength.$params.min }} حرف  <br /></span>
                                                    <span v-if="v$.code.minLength.$invalid"> {{$t('global.IsMustHaveAtMost')}}  {{ v$.code.maxLength.$params.max }} حرف</span>
                                                </div>
                                            </div>

                                            <div class="col-md-6 mb-3">
                                                <label >{{$t('global.Type')}}</label>
                                                <select
                                                    name="type"
                                                    class="form-control"
                                                    v-model="v$.type.$model"
                                                    :class="{'is-invalid':v$.type.$error,'is-valid':!v$.type.$invalid}"
                                                >
                                                    <option value="">---</option>
                                                    <option value="fixed" >{{ $t('global.Fixed') }}</option>
                                                    <option value="percentage">{{ $t('global.Percentage') }}</option>
                                                </select>
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                    <span v-if="v$.type.required.$invalid"> {{ $t('global.IsRequired') }}<br /> </span>
                                                </div>
                                            </div>

                                            <div class="col-md-6 mb-3">
                                                <label for="validationCustom03">{{$t('global.Value')}}</label>
                                                <input type="text" class="form-control"
                                                       v-model.trim.number="v$.value.$model"
                                                       id="validationCustom03"
                                                       placeholder="القيمة"
                                                       :class="{'is-invalid':v$.value.$error,'is-valid':!v$.value.$invalid}"
                                                >
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                    <span v-if="v$.value.required.$invalid"> {{ $t('global.IsRequired') }}<br /> </span>
                                                    <span v-if="v$.value.numeric.$invalid"> {{$t('global.ThisFieldIsNumeric')}} <br /> </span>
                                                </div>
                                            </div>

                                            <div class="col-md-6 mb-3">
                                                <label for="validationCustom04">عدد المستخدمين</label>
                                                <input type="text" class="form-control"
                                                       v-model.trim.number="v$.use_times.$model"
                                                       id="validationCustom04"
                                                       placeholder="عدد المستخدمين"
                                                       :class="{'is-invalid':v$.use_times.$error,'is-valid':!v$.use_times.$invalid}"
                                                >
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                    <span v-if="v$.use_times.required.$invalid"> {{ $t('global.IsRequired') }}<br /> </span>
                                                    <span v-if="v$.use_times.numeric.$invalid"> {{$t('global.ThisFieldIsNumeric')}} <br /> </span>
                                                </div>
                                            </div>

                                            <div class="col-md-6 mb-3">
                                                <label for="validationCustom06">وقت البداية</label>
                                                <input type="date" class="form-control"
                                                       v-model.trim="v$.start_date.$model"
                                                       id="validationCustom06"
                                                       :class="{'is-invalid':v$.start_date.$error,'is-valid':!v$.start_date.$invalid}"
                                                >
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                </div>
                                            </div>

                                            <div class="col-md-6 mb-3">
                                                <label for="validationCustom07">وقت النهاية</label>
                                                <input type="date" class="form-control"
                                                       v-model.trim="v$.expire_date.$model"
                                                       id="validationCustom07"
                                                       placeholder="اسم المسئول لدي المورد"
                                                       :class="{'is-invalid':v$.expire_date.$error,'is-valid':!v$.expire_date.$invalid}"
                                                >
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                    <span v-if="v$.expire_date.required.$invalid"> {{ $t('global.IsRequired') }}<br /> </span>
                                                </div>
                                            </div>

                                            <div class="col-md-6 mb-3">
                                                <label for="validationCustom09">الرقم اكثر من</label>
                                                <input type="text" class="form-control"
                                                       v-model.trim="v$.greater_than.$model"
                                                       id="validationCustom09"
                                                       placeholder="الرقم اكثر من "
                                                       :class="{'is-invalid':v$.greater_than.$error,'is-valid':!v$.greater_than.$invalid}"
                                                >
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                    <span v-if="v$.greater_than.numeric.$invalid"> {{$t('global.ThisFieldIsNumeric')}} <br /> </span>
                                                </div>
                                            </div>

                                            <div class="col-md-12 mb-3">
                                                <label for="validationCustom034">الوصف</label>
                                                <textarea type="text" class="form-control custom-textarea"
                                                       v-model.trim="v$.description.$model"
                                                       id="validationCustom034"
                                                       placeholder="الوصف"
                                                       :class="{'is-invalid':v$.description.$error,'is-valid':!v$.description.$invalid}"
                                                ></textarea>
                                                <div class="valid-feedback">{{$t('global.LooksGood')}}</div>
                                                <div class="invalid-feedback">
                                                </div>
                                            </div>


                                        </div>

                                        <button class="btn btn-primary" type="submit">{{$t('global.Submit')}}</button>
                                    </form>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <!-- /Table -->
        </div>
    </div>
</template>

<script>
import {computed, onMounted, reactive,toRefs,inject,ref} from "vue";
import useVuelidate from '@vuelidate/core';
import {required,minLength,between,maxLength,alpha,numeric} from '@vuelidate/validators';
import adminApi from "../../../api/adminAxios";
import { notify } from "@kyvg/vue3-notification";
import { useI18n } from 'vue-i18n';


export default {
    name: "createDepartment",
    data(){
        return {
            errors:{}
        }
    },
    setup(){
        let {t} = useI18n();
        let loading = ref(false);

        //start design
        let addDiscount =  reactive({
            data:{
                code : '',
                type : '',
                value : null,
                description : '',
                use_times : null,
                start_date : new Date().toISOString().split('T')[0],
                expire_date : '',
                greater_than: null
            }
        });

        const rules = computed(() => {
            return {
                code: {
                    minLength: minLength(3),
                    maxLength:maxLength(70),
                    required
                },
                type: {
                    required
                },
                value: {
                    numeric,
                    required
                },
                use_times: {
                    required,
                    numeric
                },
                expire_date: {
                    required
                },
                greater_than: {
                    numeric
                },
                start_date:{},
                description:{}
            }
        });


        const v$ = useVuelidate(rules,addDiscount.data);


        return {loading,...toRefs(addDiscount),v$,t};
    },
    methods: {
        storeDiscount(){

            this.v$.$validate();

            if(!this.v$.$error){

                this.loading = true;
                this.errors = {};

                adminApi.post(`/v1/dashboard/discount`,this.data)
                    .then((res) => {

                        notify({
                            title: `${this.t('global.AddedSuccessfully')} <i class="fas fa-check-circle"></i>`,
                            type: "success",
                            duration: 5000,
                            speed: 2000
                        });

                        this.resetForm();
                        this.$nextTick(() => { this.v$.$reset() });
                    })
                    .catch((err) => {
                        this.errors = err.response.data.errors;
                    })
                    .finally(() => {
                        this.loading = false;
                    });

            }
        },
        resetForm(){
            this.data.code = '';
            this.data.type = '';
            this.data.value = '';
            this.data.use_times = '';
            this.data.expire_date = '';
            this.data.greater_than = '';
            this.data.start_date = '';
            this.data.description = '';
        }
    }
}
</script>

<style scoped>
.coustom-select {
    height: 100px;
}
.card{
    position: relative;
}

.custom-textarea {
    height: 150px;
}
</style>
