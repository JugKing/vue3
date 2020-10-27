<!-- 系统登录组件 -->
<template>
    <div class="login">
        <a-spin key="loginLoading" :spinning="$store.state.menuLoading" size="large">
            <div class="login-header">
                <div class="logo">
                    <img src="../assets/icon_logo.svg" alt="" />
                    <div class="title">{{ $t("Utils.webstitle") }}</div>
                </div>
            </div>
            <!-- <div class="login-content">
                    <div class="box">
                        <div class="title">{{$t('Login.title')}}</div>
                        <div class="login-mes">{{loginMessage}}</div>
                        <div class="login-form">
                            <a-input class="ani-input1" :placeholder="$t('Login.placeName')" size="large" v-model="params.userName" @change="e=>inputChange(e,'userName')" maxLength="32" />
                            <a-input class="ani-input2" :placeholder="$t('Login.placePwd')" size="large" v-model="params.password" @change="e=>inputChange(e,'password')" maxLength="32" type="password" />
                            <div class="ani-input3 ver-code">
                                <a-input :placeholder="$t('Login.placeCode')" v-model="params.vertical" @change="e=>inputChange(e,'vertical')" maxLength="6" size="large" />
                                <div class="vertical">
                                    <img src="" alt="">
                                </div>
                            </div>
                            <a-checkbox v-model="params.remember" class="ani-input4 remember-me">
                                {{$t('Login.rememberMe')}}
                            </a-checkbox>
                            <a-button class="ani-input5 login-btn" @click="submitFn" :loading="loading">
                                {{$t('Login.login')}}
                            </a-button>
                        </div>
                    </div>
                </div> -->
            <transition name="fade">
                <a-spin key="loginLoading" :spinning="loading" size="large">
                    <div class="login-content" v-if="isexist">
                        <h3 class="login-title">智能贷款 - 选择机构</h3>
                        <div class="form">
                            <div class="form-item">
                                <div class="label">
                                    <span>*</span>选择机构：
                                </div>
                                <div class="item">
                                    <a-select style="width:240px;" @change="selectOrgItem" v-model="orgNo" placeholder="请选择所属机构">
                                        <a-select-option v-for="(item, index) in orgList" :key="`orgNo-${index}`" :value="item.orgNo">{{ item.orgNm }}</a-select-option>
                                    </a-select>
                                </div>
                            </div>
                            <!-- <div class="btn-box">
                        <a-button type="primary">提交</a-button>
                    </div> -->
                        </div>
                    </div>
                    <div class="login-content" v-else>
                        <h3 class="login-title">智能贷款 - 选择机构</h3>
                        <div class="error-login" style="text-align:center;">
                            服务器开小差了，请联系系统管理员或技术人员!
                        </div>
                    </div>
                </a-spin>
            </transition>
            <div class="login-footer">
                <img src="@assets/login/bg1.png" class="translateY" />
                <img src="@assets/login/bg2.png" class="translateY" />
                <img src="@assets/login/bg3.png" class="translateY2" />
                <img src="@assets/login/bg4.png" class="translateY1" />
                <img src="@assets/login/bg5.png" class="star" />
                <div class="info op">{{ $t("Login.copyright") }}</div>
            </div>
        </a-spin>
    </div>
</template>

<script>
import { message } from 'ant-design-vue';
import apis from '@network/api';
import { Modal } from 'ant-design-vue';

export default {
    name: 'login',
    data: () => {
        return {
            loading: true,
            loginMessage: '',
            loginFlag: false,
            params: {},
            sidId: '', // 门户网站返回的id
            search: '',

            // 默认显示机构选择
            isexist: true,
            // 默认隐藏机构选择
            // isexist: false,
            orgList: [],
            orgNo: '', // 登录用户选择的机构,
            loginSearch: '',
        };
    },
    //进入登录页面后保存sid，重置地址栏
    // beforeCreate() {
    //     // const search = window.location.search;
    //     // sessionStorage.setItem('search', search);
    //     // window.location.search = '';
    // },
    created() {
        // 记住密码数据还原
        if (localStorage.getItem('userInfo')) {
            
            const userInfo = JSON.parse(localStorage.getItem('userInfo')) || {};
            userInfo.remember && Object.assign(this.params, userInfo);
        }

        this.loginSearch = window.location.search;
        if (!sessionStorage.getItem('loginSearch')) {
            sessionStorage.setItem(
                'loginSearch',
                JSON.stringify([this.loginSearch])
            );
        } else {
            if (
                this.loginSearch &&
                JSON.parse(sessionStorage.getItem('loginSearch'))[0] !=
                    this.loginSearch
            )
                sessionStorage.setItem(
                    'loginSearch',
                    JSON.stringify([this.loginSearch])
                );
        }

        var refurlTest = document.referrer;

        // let refurlTest='http://10.0.193.91:8001/portal/server.pt?CommunityID=224&parentname=Login&control=SetCommunity&parentid=1&in_hi_userid=313301&PageID=0&cached=false&space=CommunityPage';
        console.log(refurlTest);
        let secRefurl = window.btoa('refurl');
        if (refurlTest && !sessionStorage.getItem(`${secRefurl}`)) {
            sessionStorage.setItem(`${secRefurl}`, window.btoa(refurlTest));
        } else {
            if (sessionStorage.getItem(`${secRefurl}`)) {
                refurlTest = window.atob(
                    sessionStorage.getItem(`${secRefurl}`)
                );
            }
        }
        if (!refurlTest) {
            if (sessionStorage.getItem(`${secRefurl}`))
                refurlTest = window.atob(
                    sessionStorage.getItem(`${secRefurl}`)
                );
        }
        console.log('网站来源', refurlTest);
        //dev-10.16.0.219  sit-10.0.193.91
        // var aSites = new Array('10.16.0.219', '10.0.193.91', 'localhost');
        if (process.env.SITES == 'false') {
            this.getGatewayParams();
        } else {
            var aSites = process.env.SITES.split(',');
            var fromEmpCen = false;
            for (let i in aSites) {
                if (refurlTest && refurlTest.indexOf(aSites[i]) > 0) {
                    fromEmpCen = true;
                    break;
                }
            }
            if (fromEmpCen) {
                this.getGatewayParams();
            } else {
                Modal.warning({
                    title: '提示',
                    content:
                        '您的访问来源非员工门户，请登录员工门户后再访问系统！',
                });
            }
        }
    },

    methods: {
        // submitFn() {
        //     if (process.env.RUN_TYPE === 'alibeta') {
        //         this.$router.loginRouter();
        //         return false;
        //     }
        //     this.validatorLogin();
        //     if (!this.loginFlag) return;
        //     this.loading = true;
        //     apis.login({
        //         UserId: this.params.userName || ``,
        //         Password: this.params.password || ''
        //     }).then(
        //         res => {
        //             this.loading = false;
        //             this.rememberMe();
        //             const { data } = res;
        //             this.$store.commit('setUserInfo', data);
        //             // 设置用户所拥有的权限路由
        //             this.$router.loginRouter();
        //         },
        //         err => {
        //             this.loading = false;
        //             this.loginMessage =
        //                 err.message || this.$t('Login.loginFailure');
        //         }
        //     );
        // },

        // inputChange(e, type) {
        //     // 做验证处理
        //     // const { userName, password, vertical } = this.params;
        //     const v = e.target.value;
        //     switch (type) {
        //         case 'userName':
        //             this.loginMessage = !v
        //                 ? this.$t('Login.verification.user')
        //                 : '';
        //             break;
        //         case 'password':
        //             if (v.length < 7) {
        //                 this.loginMessage = this.$t(
        //                     'Login.verification.pwdLength'
        //                 );
        //                 return false;
        //             }
        //             this.loginMessage = this.regs.reg01.test(v)
        //                 ? ''
        //                 : this.$t('Login.verification.pwdType');
        //             break;
        //         case 'vertical':
        //             this.loginMessage = !v
        //                 ? this.$t('Login.verification.vertical')
        //                 : '';
        //             break;
        //     }
        // },

        //    rememberMe() {
        //         const userInfo = {
        //             remember: this.params.remember || false,
        //             userName: this.params.userName || '',
        //             password: this.params.password || ''
        //         };
        //         localStorage.setItem('userInfo', JSON.stringify(userInfo));
        //     },
        validatorLogin() {
            const params = this.params;
            if (!params.userName) {
                this.loginMessage = this.$t('Login.verification.user');
            } else if (!params.password) {
                this.loginMessage = this.$t('Login.verification.pwd');
            } else if (!params.vertical) {
                this.loginMessage = this.$t('Login.verification.vertical');
            }
            this.loginFlag =
                params.userName && params.password && params.vertical;
        },
        // 第三方登录
        getGatewayParams() {
            let localSearch = window.location.search;
            let search;

            if (localSearch) {
                search = localSearch;
            } else {
                try {
                    this.loginSearchTmp = JSON.parse(
                        sessionStorage.getItem('loginSearch')
                    )[0];
                    search = this.loginSearchTmp;
                } catch (error) {
                    error;
                }
            }

            // console.log(search);

            if (!search) {
                try {
                    search = JSON.parse(sessionStorage.getItem('loginSidId'));
                } catch (error) {
                    error;
                }
            }

            if (search && search.indexOf('=') != -1) {
                // const regExp = new RegExp('(?<=sid=).+&?', 'g');
                // 获取浏览器的SID参数，ie不支持此正则
                // this.sidId = window.location.search.match(regExp)[0];

                // 员工中心进入角色权限设置
                let beforeHandleSidId = '';
                if (search.indexOf('&') != -1) {
                    beforeHandleSidId = search.split('?')[1].split('&');
                    if (beforeHandleSidId.length) {
                        beforeHandleSidId.forEach((item) => {
                            if (item.includes('sid')) {
                                this.sidId = item.split('=')[1];
                            }
                            // 通过员工中心传值判断是否进入角色权限设置页面
                            if (item.includes('roleSet')) {
                                let tempRoleSet = item.split('=')[1];
                                // 传递值为true则存true 不传或不为true则存false
                                if (tempRoleSet == 'true') {
                                    sessionStorage.setItem('fromEmpCt', 'true');
                                } else {
                                    sessionStorage.setItem(
                                        'fromEmpCt',
                                        'false'
                                    );
                                }
                            }
                        });
                    }
                } else {
                    this.sidId = search.split('=')[1];
                    sessionStorage.setItem('fromEmpCt', 'false');
                }
            } else {
                let tempSidId = JSON.parse(
                    sessionStorage.getItem('loginSidId')
                );

                this.sidId = tempSidId;
            }

            sessionStorage.setItem(`loginSidId`, JSON.stringify(this.sidId));
            // 本地开发测试代码
            if (process.env.RUN_TYPE === 'alibeta') {
                this.$router.loginRouter();

                return false;
            }
            if (!this.sidId) {
                message.warning(`登录sidId不存在或登录入口发生错误，请检查！`);
                return;
            }

            // 访问后端接口，判断此用户是否存在多个机构号
            this.sidId && this.getUserInfoByGateway({ sidId: this.sidId });
        },
        // 访问后端接口
        getUserInfoByGateway(params) {
            this.loading = true;

            apis.gatewayLogin(params).then(
                (res) => {
                    if (res.orgList && res.orgList.length > 1) {
                        // 存在多个机构信息
                        this.orgList = res.orgList;
                        this.isexist = true;
                    }

                    if (res.token || res.loginOrgNm) {
                        const userInfo = {
                            userName: res.emplyName || '',
                            ...res,
                        };
                        localStorage.setItem(
                            'userInfo',
                            JSON.stringify(userInfo)
                        );

                        this.$store.commit('setUserInfo', userInfo);
                        // 用户选择机构后直接跳转到首页home
                        this.$router.loginRouter();
                    }
                    this.loading = false;
                },
                (err) => {
                    if (err.status == 502) {
                        message.error('网关错误，登录失败！');
                    }
                    // retMsgCode: "SMERR00009"
                    // retMessage: "请求员工中心失败"
                    if (err && err.retMsgCode) {
                        message.error(err.retMessage);
                    }
                    this.isexist = false;
                    this.loading = false;
                }
            );
        },
        // 切换机构账号后重新请求登录接口
        selectOrgItem(e = '') {
            let tempOrgNo = this.$store.state.userInfo.loginOrgNo;
            // zz代码
            if (tempOrgNo) {
                if (tempOrgNo == e) {
                    this.getUserInfoByGateway({
                        loginOrgNo: e,
                        sidId: this.sidId,
                    });
                } else {
                    this.getUserInfoByGateway({
                        loginOrgNo: e,
                        sidId: this.sidId,
                    });
                }
            } else {
                this.getUserInfoByGateway({ loginOrgNo: e, sidId: this.sidId });
            }
        },
    },
};
</script>
<style lang="less">
.login {
    .ant-spin-container {
        height: 100vh;
    }

    .ant-checkbox-checked .ant-checkbox-inner {
        background-color: #1da0ce;
        border-color: #fff;
    }

    .ant-checkbox-wrapper:hover .ant-checkbox-inner,
    .ant-checkbox:hover .ant-checkbox-inner,
    .ant-checkbox-input:focus + .ant-checkbox-inner {
        border-color: #fff;
    }

    .ant-checkbox-inner {
        background: transparent;
    }
    .item .ant-select-selection {
        background-color: rgba(0, 57, 71, 0);
        border: 1px solid #00a9dd;
        color: #29cdff;
    }
    .item .ant-select-arrow-icon {
        color: #29cdff;
    }
}
</style>
<style scoped lang="less">
.login {
    width: 100%;
    height: 100vh;
    position: relative;
    overflow: hidden;
    background-color: #005771;

    .logo {
        width: 980px;
        height: 50px;
        font-size: 18px;
        padding: 0 20px;
        margin: 0 auto;
        display: flex;
        align-items: center;
        color: #00dcff;

        img {
            display: inline-block;
            height: 32px;
            margin-right: 10px;
            vertical-align: middle;
        }
    }
}

.login-loading {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
}

.login-content {
    width: 420px;
    height: 240px;
    position: relative;
    padding: 20px 40px;
    z-index: 2;
    /*margin: ~"calc(50vh - 260px)" auto 0;*/
    color: #29cdff;
    margin: ~'calc(10vh)' auto 0;
    border-radius: 10px;
    box-shadow: 1px -3px 6px #007fa5, 1px -3px 6px #007fa5;

    h3.login-title {
        color: #29cdff;
        text-align: center;
    }
    .form {
        text-align: center;
    }
    .form-item {
        display: flex;
        margin: 50px auto;
        align-items: center;
        .label {
            min-width: 85px;
            span {
                color: #f5222d;
                margin-right: 6px;
            }
        }
        .item {
            max-width: 215px;
            flex: 1;
        }
    }
    .login-mes {
        height: 20px;
        line-height: 20px;
        margin: 5px 0 10px;
        color: #df6704;
    }

    .box {
        padding: 20px;
        /*transition: all 1.2s;*/

        /*&:hover {*/
        /*    background-color: #005771;*/
        /*    border-radius: 10px;*/
        /*    box-shadow: 1px 0 18px #aaa, 1px 0 18px #aaa;*/
        /*}*/

        .title {
            text-align: center;
            font-size: 20px;
            color: #fff;
            margin-bottom: 20px;
        }
    }

    input {
        font-size: 14px;
        background-color: #006e95;
        border: none;
        color: #9dd7ea;

        &::placeholder {
            color: #9dd7ea;
        }

        &:focus {
            border: none;

            &::placeholder {
                color: #f2f2f2;
            }
        }
    }

    input + input {
        margin-top: 20px;
    }

    .ver-code {
        margin-bottom: 20px;
        display: flex;
        justify-content: space-between;
    }

    .vertical {
        width: 100px;
        height: 40px;
        margin-left: 14px;
        border-radius: 4px;
        background: repeating-linear-gradient(
            140deg,
            #1da0ce,
            #1da0ce 2px,
            #006e95 2px,
            #006e95 8px
        );
    }

    .remember-me {
        color: #fff;
        margin-top: 20px;
        border-color: #fff;
    }

    .login-btn {
        display: block;
        margin: 40px 0 30px;
        height: 40px;
        line-height: 40px;
        width: 100%;
        color: #fff;
        border-color: #1ae1d1;
        background: #1ae1d1;
    }
}

.ani-input1 {
    animation: input-ani 2s;
}

.ani-input2 {
    animation: input-ani 1.8s 0.1s;
}

.ani-input3 {
    animation: input-ani 1.6s 0.2s;
}

.ani-input4 {
    animation: input-ani 1.4s 0.3s;
}

.ani-input5 {
    animation: input-ani 1.2s 0.4s;
}

.ani-input1,
.ani-input2,
.ani-input3,
.ani-input4,
.ani-input5 {
    opacity: 0;
    animation-fill-mode: forwards;
}

@keyframes input-ani {
    0% {
        opacity: 0.2;
        transform: translateY(60px);
    }
    100% {
        opacity: 1;
        transform: translateY(0px);
    }
}

.login-footer {
    height: 100%;
    bottom: 0;

    img {
        width: 100%;
        z-index: 1;
        position: absolute;
        bottom: 0;
        left: 0;
    }

    .star {
        bottom: 28%;
        opacity: 0;
        animation: star-ain 1.5s 1s;
        animation-fill-mode: forwards;
    }

    @keyframes star-ain {
        from {
            transform: scale(0.1, 0.1) translateY(200px);
        }

        to {
            opacity: 1;
            transform: scale(1, 1) translateY(0px);
        }
    }

    .info {
        position: absolute;
        bottom: 0;
        z-index: 1;
        width: 100%;
        text-align: center;
        padding: 20px 0 30px;
        color: #717a88;
        opacity: 0.5;
        animation: info-ain 2s;
        animation-fill-mode: forwards;
    }

    @keyframes info-ain {
        from {
            opacity: 0.5;
        }
        to {
            opacity: 1;
        }
    }
}

.translateY,
.translateY1,
.translateY1 {
    animation: translateY 2s;
    opacity: 0.2;
    transform: translateY(-60px) scale(1, 2);
    animation-fill-mode: forwards;
}

.translateY1 {
    animation: translateY1 1s;
    animation-fill-mode: forwards;
}

.translateY2 {
    animation: translateY 3s;
    animation-fill-mode: forwards;
}

@keyframes translateY {
    0% {
        opacity: 0.2;
        transform: translateY(-60px) scale(1, 2);
    }
    30% {
        opacity: 0.6;
        transform: translateY(10px) scale(1, 1);
    }
    100% {
        opacity: 1;
        transform: translateY(0px);
    }
}

@keyframes translateY1 {
    0% {
        opacity: 0.2;
        transform: translateY(-60px) scale(1, 2);
    }
    100% {
        opacity: 1;
        transform: translateY(0px) scale(1, 1);
    }
}

@keyframes translateY2 {
    0% {
        opacity: 0.2;
        transform: translateY(-60px) scale(1, 2);
    }
    30% {
        opacity: 0.6;
        transform: translateY(10px) scale(1, 1);
    }
    100% {
        opacity: 1;
        transform: translateY(0px) scale(1, 1);
    }
}
</style>
