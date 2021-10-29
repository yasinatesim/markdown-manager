<template>
  <button
    :class="'btn-' + variant"
  >
    <slot />
  </button>
</template>

<script>
export default {
	props: ['variant']
}
</script>