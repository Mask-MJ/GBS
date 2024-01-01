<template>
<section class="content">
  <div>
      <router-view @show-session-list="(filter, sms) => $emit('show-session-list', filter, sms)"></router-view>
  </div>
</section>
</template>

<script>
export default {
  data(){
      return {
      }
  }
}
</script>
