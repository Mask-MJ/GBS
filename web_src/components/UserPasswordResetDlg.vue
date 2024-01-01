<template>
    <FormDlg :title="title" @hide="onHide" @show="onShow" @submit="onSubmit" ref="dlg" :disabled="errors.any()">
        <div :class="{'form-group':true, 'has-feedback':true,'has-error': errors.has('newPassword')}">
            <label for="new-password" class="col-sm-4 control-label">新密码
                <span class="text-red">*</span>
            </label>
            <div class="col-sm-7">
                <input type="password" class="form-control" name="newPassword" v-model.trim="form.newPassword" autocomplete="new-password" data-vv-as="新密码" v-validate="'required'" @keydown.enter="onSubmit">
                <p class="help-block" style="margin-bottom:0;" v-show="form.newPassword">
                    <PasswordLine ref="pwd-line" :pwd="form.newPassword" @update:level="level = $event;"/>
                </p>
            </div>
        </div>
    </FormDlg>
</template>

<script>
import { mapState, mapActions } from "vuex";
import FormDlg from 'components/FormDlg.vue';
import $ from 'jquery';
import PasswordLine from 'components/PasswordLine.vue';
import { JSEncrypt } from 'jsencrypt';

export default {
    props: {
        pwdLevel: {
            type: Number,
            default: 0,
        }
    },
    data() {
        return {
            form: this.defForm(),
            username: "",
            level: 0,
        }
    },
    components: {
        FormDlg, PasswordLine
    },
    computed: {
        ...mapState(["serverInfo"]),
        title() {
            if(this.username) {
                return `重置密码(${this.username})`;
            }
            return "重置密码";
        }
    },
    methods: {
        ...mapActions(["getServerInfo"]),
        defForm() {
            return {
                uid: '',
                newPassword: '',
            }
        },
        onHide() {
            this.form = this.defForm();
            this.level = 0;
            this.username = "";
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
            $.get('/api/v1/user/resetpassword', {
                id: this.form.uid,
                password: this.encryptPwd(this.form.newPassword),
            }).then(data => {
                this.$refs['dlg'].hide();
                this.$message({
                    type: 'success',
                    message: "密码重置成功！"
                })
            })
        },
        show(id, username = "") {
            this.errors.clear();
            this.form.uid = id;
            this.username = username || "";
            this.$refs['dlg'].show();
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
