<template>
  <div>
    <NuxtLink :to="'/admin/dashboard/products'">
      products
    </NuxtLink>
    |
    <NuxtLink :to="'/admin/dashboard/orders'">
      orders
    </NuxtLink>
    |
    <NuxtLink :to="'/admin/dashboard/coupons'">
      coupons
    </NuxtLink>
    <div>
      <NuxtChild :key="$route.params.id" />
    </div>
  </div>
</template>

<script>
export default {
  mounted () {
    const token = document.cookie.replace(
      /(?:(?:^|.*;\s*)hexToken\s*=\s*([^;]*).*$)|^.*$/,
      '$1'
    )
    this.$axios.defaults.headers.common.Authorization = `${token}`
  }
}
</script>
