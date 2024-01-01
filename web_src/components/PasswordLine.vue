<template>
    <FormatProgress text-inside :stroke-width="height"
        :percentage="percentage" :status="status" :format="format">
    </FormatProgress>
</template>

<script>
import FormatProgress from 'components/FormatProgress.vue'

export default {
    props: {
        pwd: {
            type: String,
            default: "",
        },
        height: {
            type: Number,
            default: 18,
        }
    },
    components: {
        FormatProgress
    },
    computed: {
        level() {
            var l = 0;
            if(!this.pwd) {
                this.$emit("update:level", l);
                return l;
            }
            if(this.pwd.length >= 1) l++;
            if(this.isWeakPassword(this.pwd)) {
                this.$emit("update:level", l);
                return l;
            }
            if(this.pwd.length >= 4) l++;
            if(this.pwd.length >= 6) {
                if(/\d/.test(this.pwd)) l++; // 数字
                if(/[a-z]/.test(this.pwd)) l++; // 小写
                if(/[A-Z]/.test(this.pwd)) l++; // 大写
                if(/\W/.test(this.pwd)) l++; // 特殊字符
                if(!/(\w)\1/.test(this.pwd)) l++; // 连续字符
            }
            if(this.pwd.length >= 8) l++;
            if(this.pwd.length >= 12) l++;
            this.$emit("update:level", l);
            return l;
        },
        status() {
            if(this.percentage >= 90) { // level >= 6
                return "success";
            }
            if(this.percentage >= 45) { // level >= 3
                return "warning";
            }
            return "exception";
        },
        percentage() {
            var ret = this.level * 15;
            if(ret < 0) ret = 0;
            if(ret > 100) ret = 100;
            return ret;
        }
    },
    methods: {
        format(percentage) {
            if(percentage >= 90) { // level >= 6
                return "强";
            }
            if(percentage >= 45) { // level >= 3
                return "中";
            }
            return "弱";
        },
        isWeakPassword(password) {
            if(password.length < 3) return true;
            var numberChars = '0123456789';
            var lowerChars = 'abcdefghijklmnopqrstuvwxyz';
            var upperChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
            var numberWeak = false;
            var lowerWeak = false;
            var upperWeak = false;

            for (var i = 0; i < password.length - 2; i++) {
                var start = numberChars.indexOf(password[i]);
                var middle = numberChars.indexOf(password[i + 1]);
                var end = numberChars.indexOf(password[i + 2]);
                if (start >= 0 && middle >= 0 && end >= 0) {
                    if ((end === middle + 1 && middle === start + 1) || (end == middle && middle == start)) {
                        numberWeak = true;
                        continue
                    }
                }
                numberWeak = false;
                break;
            }
            if(numberWeak) return true;

            for (var i = 0; i < password.length - 2; i++) {
                var start = lowerChars.indexOf(password[i]);
                var middle = lowerChars.indexOf(password[i + 1]);
                var end = lowerChars.indexOf(password[i + 2]);
                if (start >= 0 && middle >= 0 && end >= 0) {
                    if ((end === middle + 1 && middle === start + 1) || (end == middle && middle == start)) {
                        lowerWeak = true;
                        continue
                    }
                }
                lowerWeak = false;
                break;
            }
            if(lowerWeak) return true;

            for (var i = 0; i < password.length - 2; i++) {
                var start = upperChars.indexOf(password[i]);
                var middle = upperChars.indexOf(password[i + 1]);
                var end = upperChars.indexOf(password[i + 2]);
                if (start >= 0 && middle >= 0 && end >= 0) {
                    if ((end === middle + 1 && middle === start + 1) || (end == middle && middle == start)) {
                        upperWeak = true;
                        continue
                    }
                }
                upperWeak = false;
                break;
            }
            if(upperWeak) return true;
            return false;
        }
    }
}
</script>
