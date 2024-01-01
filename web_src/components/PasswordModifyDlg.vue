<template>
    <FormDlg :title="title" @hide="onHide" @show="onShow" @submit="onSubmit" ref="dlg" :disabled="errors.any()" :closeable="!pwdModReq">
        <div :class="{'form-group':true, 'has-feedback':true,'has-error': errors.has('oldPassword')}" v-if="!pwdModReq">
            <label for="old-password" class="col-sm-4 control-label">原密码
                <span class="text-red">*</span>
            </label>
            <div class="col-sm-7">
                <input type="password" class="form-control" id="old-password" name="oldPassword" v-model.trim="form.oldPassword" autocomplete="new-password" data-vv-as="原密码" v-validate="'required'" @keydown.enter="$el.querySelector('#new-password').focus()">
            </div>
        </div>
        <div :class="{'form-group':true, 'has-feedback':true,'has-error': errors.has('newPassword')}">
            <label for="new-password" class="col-sm-4 control-label">新密码
                <span class="text-red">*</span>
            </label>
            <div class="col-sm-7">
                <input type="password" class="form-control" id="new-password" name="newPassword" v-model.trim="form.newPassword" autocomplete="new-password" data-vv-as="新密码" v-validate="'required'" @keydown.enter="$el.querySelector('#new-password2').focus()">
                <p class="help-block" style="margin-bottom:0;" v-show="form.newPassword">
                    <PasswordLine ref="pwd-line" :pwd="form.newPassword" @update:level="level = $event;"/>
                </p>
            </div>
        </div>
        <div :class="{'form-group':true, 'has-feedback':true,'has-error': errors.has('newPassword2')}">
            <label for="new-password2" class="col-sm-4 control-label">确认密码
                <span class="text-red">*</span>
            </label>
            <div class="col-sm-7">
                <input type="password" class="form-control" id="new-password2" name="newPassword2" v-model.trim="form.newPassword2" autocomplete="new-password" data-vv-as="确认密码" v-validate="'required|confirmed:newPassword'" @keydown.enter="onSubmit">
            </div>
        </div>
        <template slot="footer" v-if="pwdModReq">
            <a role="button" class="btn btn-default" href="/logout">退出登录</a>
        </template>
    </FormDlg>
</template>

<script>
import FormDlg from 'components/FormDlg.vue';
import PasswordLine from 'components/PasswordLine.vue';
import { mapState, mapActions } from "vuex";
import $ from 'jquery';
import crypto from 'crypto';
import { JSEncrypt } from "jsencrypt";

export default {
    props: {
        pwdLevel: {
            type: Number,
            default: 3,
        },
        pwdModReq: {
            type: Boolean,
            default: false,
        }
    },
    data() {
        return {
            form: this.defForm(),
            level: 0,
        }
    },
    components: {
        FormDlg, PasswordLine
    },
    computed: {
        ...mapState(["serverInfo"]),
        title() {
            if(this.pwdModReq) {
                return "设置安全登录密码";
            }
            return "修改密码";
        },
    },
    methods: {
        ...mapActions(["getServerInfo"]),
        defForm() {
            return {
                oldPassword: '',
                newPassword: '',
                newPassword2: ''
            }
        },
        onHide() {
            this.form = this.defForm();
            this.level = 0;
        },
        onShow() {
            this.errors.clear();
            // this.$el.querySelector('#old-password').focus();
        },
        async onSubmit() {
            var ok = await this.$validator.validateAll();
            if(!ok) {
                var e = this.errors.items[0];
                this.$message({
                    type: 'error',
                    message: e.msg
                })
                $(`[name=${e.field}]`).focus();
                return;
            }
            if(this.level < this.pwdLevel) {
                this.$message({
                    type: 'error',
                    message: "密码强度弱",
                })
                return
            }
            $.get('/api/v1/modifypassword', {
                oldpassword: this.encryptPwd(this.md5(this.form.oldPassword)),
                newpassword: this.encryptPwd(this.md5(this.form.newPassword))
            }).then(data => {
                this.$refs['dlg'].hide();
                this.$alert("密码修改成功,即将重新登录!", "提示").then(() => {
                    window.location.href = "/logout";
                }).catch(() => {
                    window.location.href = "/logout";
                })
            })
        },
        show(data) {
            this.errors.clear();
            if(data) {
                Object.assign(this.form, data);
            }
            this.$refs['dlg'].show();
        },
        md5(text) {
            return crypto.createHash('md5').update(text, "utf8").digest('hex');
        },
        encryptPwd(password) {
            if(!this.serverInfo.PwdPub) return password;
            var pubKey = window.atob(this.serverInfo.PwdPub);
            if(pubKey.indexOf("RSA Public Key") < 0) return password;
            var encrypt = new JSEncrypt();
            encrypt.setPublicKey(pubKey);
            return encrypt.encrypt(password);
        },
    }
}
</script>
