<script lang="coffee">
export default
  props:
    close: Function
</script>

<template lang="pug">
v-btn.dismiss-modal-button(icon, :aria-label="$t('common.action.cancel')", @click='close()')
  v-icon mdi-window-close
</template>
