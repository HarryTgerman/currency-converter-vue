<template>
  <div class="switch-button-container" @click="click">
    <div class="switch-button-inner">
      <i class="fas fa-sync"></i>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    click: Function,
  },
};
</script>
