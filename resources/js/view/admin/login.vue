<template>
    <!-- Main Wrapper -->
    <div class="main-wrapper login-body">

        <loader v-if="loading" />

        <div class="login-wrapper">
            <div class="container">
                <a class="download" download href="/web/mandob.apk">{{$t('global.downloadRepresentative')}}</a>
                <img class="img-fluid logo-dark mb-2" src="/web/img/logo.png" alt="Logo">
                <div class="loginbox">

                    <div class="login-right">
                        <div class="login-right-wrap">
                            <h1>{{ $t("global.welcomeBack") }}</h1>
                            <p class="account-subtitle">{{ $t("global.DontmissyournextopportunitySigninto") }}</p>

                            <form @submit.prevent='Loginsubmit'>
                                <div class="form-group form-focus">
                                    <input type="text" v-model="data.email"  class="form-control floating">
                                    <label class="focus-label">{{$t('global.Email')}}</label>
                                </div>
                                <div class="form-group form-focus">
                                    <input type="password" v-model="data.password" class="form-control floating">
                                    <label class="focus-label">{{$t('global.Password')}}</label>
                                </div>
                                <div class="form-group">
                                    <label class="custom_check">
                                        <input type="checkbox" v-model="data.remmeber_me">
                                        <span class="checkmark"></span> {{ $t("global.Rememberpassword") }}
                                    </label>
                                </div>
                                <button class="btn btn-primary btn-block btn-lg login-btn" type="submit">{{ $t("global.login") }}</button>

<!--                                <div class="row form-row login-foot">-->
<!--                                    <div class="col-lg-6 login-forgot">-->
<!--                                        <router-link class="forgot-link" :to="{name:'forgetPassword'}">Forgot Password ?</router-link>-->
<!--                                    </div>-->
<!--                                </div>-->
                            </form>

                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <!-- /Main Wrapper -->
</template>

<script>
import {useStore} from "vuex";
import {computed, reactive, toRefs} from "vue";

export default {
    name: "login",
    setup(){
        const store = useStore();

        let loading = computed(() => {
            return store.getters['authAdmin/loading'];
        });

        //start design
        let login =  reactive({
            data:{
                email:'',
                password:'',
                remmeber_me: false
            }
        });

        function Loginsubmit (){
            store.dispatch('authAdmin/login',login.data);
        }

        return {Loginsubmit,...toRefs(login),loading};

    },
}
</script>

<style scoped>
.download{
    border: 2px solid;
    padding: 5px 10px;
    font-size: 16px;
    font-weight: 900;
    border-radius: 10px;
}
.download:hover{
    background-color: #cd1f21;
    color: #000000;
}
</style>
