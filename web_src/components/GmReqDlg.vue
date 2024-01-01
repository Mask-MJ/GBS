<template>
    <FormDlg title="创建国密证书请求" ref="dlg" @hide="onHide" @show="onShow" @submit="onSubmit" :disabled="errors.any()">
        <div :class="{'form-group': true, 'has-feedback': true,'has-error': errors.has('common_name')}" v-show="ctrl && alt">
            <label class="col-sm-4 control-label" title="CommonName">设备标识
                <!-- <span class="text-red">*</span> -->
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" ref="gm-common-name" name="common_name" placeholder="CommonName"
                    v-model.trim="form.common_name" data-vv-as="设备标识" v-validate="" @keydown.enter="$refs['gm-req-country'].focus()">
            </div>
        </div>
        <div :class="{'form-group': true, 'has-feedback': true, 'has-error': errors.has('country')}">
            <label class="col-sm-4 control-label">国家
                <span class="text-red">*</span>
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" ref="gm-req-country" name="country" placeholder="两字符国家代码, 如 CN"
                    v-model.trim="form.country" data-vv-as="国家" v-validate="'required|regex:^[A-Z]{2}$'" @keydown.enter="$refs['gm-req-province'].focus()">
            </div>
        </div>
        <div :class="{'form-group': true, 'has-feedback': true, 'has-error': errors.has('province')}">
            <label class="col-sm-4 control-label">省/州
                <!-- <span class="text-red">*</span> -->
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" ref="gm-req-province" name="province"
                    v-model.trim="form.province" data-vv-as="省/州" v-validate="" @keydown.enter="$refs['gm-req-locality'].focus()">
            </div>
        </div>
        <div :class="{'form-group': true, 'has-feedback': true, 'has-error': errors.has('locality')}">
            <label class="col-sm-4 control-label">地区
                <!-- <span class="text-red">*</span> -->
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" ref="gm-req-locality" name="locality"
                    v-model.trim="form.locality" data-vv-as="地区" v-validate="" @keydown.enter="$refs['gm-req-organization'].focus()">
            </div>
        </div>
        <div :class="{'form-group': true, 'has-feedback': true, 'has-error': errors.has('organization')}">
            <label class="col-sm-4 control-label">组织
                <!-- <span class="text-red">*</span> -->
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" ref="gm-req-organization" name="organization"
                    v-model.trim="form.organization" data-vv-as="组织" v-validate="" @keydown.enter="$refs['gm-req-organizational-unit'].focus()">
            </div>
        </div>
        <div :class="{'form-group': true, 'has-feedback': true, 'has-error': errors.has('organizational_unit')}">
            <label class="col-sm-4 control-label">单位
                <!-- <span class="text-red">*</span> -->
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" ref="gm-req-organizational-unit" name="organizational_unit"
                    v-model.trim="form.organizational_unit" data-vv-as="单位" v-validate="" @keydown.enter="$refs['gm-req-email'].focus()">
            </div>
        </div>
        <div :class="{'form-group': true, 'has-feedback': true, 'has-error': errors.has('email')}">
            <label class="col-sm-4 control-label">邮件
                <!-- <span class="text-red">*</span> -->
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" ref="gm-req-email" name="email"
                    v-model.trim="form.email" data-vv-as="邮件" v-validate="'email'" @keydown.enter="onSubmit">
            </div>
        </div>
    </FormDlg>
</template>

<script>
import FormDlg from 'components/FormDlg.vue'
import $ from 'jquery'

export default {
    data() {
        return {
            ctrl: false,
            alt: false,
            form: this.defForm(),
        }
    },
    components: {
        FormDlg
    },
    methods: {
        defForm() {
            return {
                common_name: "",
                country: "",
                province: "",
                locality: "",
                organization: "",
                organizational_unit: "",
                email: "",
            }
        },
        onHide() {
            this.ctrl = false;
            this.alt = false;
            this.form = this.defForm();
            $(this.$el).off("keydown", this.keyDown).off("keyup", this.keyUp);
        },
        onShow() {
            this.errors.clear();
            $(this.$el).on("keydown", this.keyDown).on("keyup", this.keyUp);
        },
        keyDown(e) {
            if(e.keyCode == 17) {
                this.ctrl = true;
            }
            if(e.keyCode == 18) {
                this.alt = true;
            }
        },
        keyUp(e) {
            // if(e.keyCode == 17) {
            //     this.ctrl = false;
            // }
            // if(e.keyCode == 18) {
            //     this.alt = false;
            // }
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
            $.get("/api/v1/gm/req/create", this.form).then(() => {
                this.$refs['dlg'].hide();
                this.$emit("submit");
            })
        },
        show() {
            this.errors.clear();
            this.$refs['dlg'].show();
        },
    }
}
</script>
