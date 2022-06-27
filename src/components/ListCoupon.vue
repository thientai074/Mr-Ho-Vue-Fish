<template>
  <div v-if="couponArray.length === 3" id="list-coupon">
    <ul class="list-group">
      <li class="list-group-item active" aria-current="true">
        Mã coupon hoặc quà Quý khách đã giựt được
      </li>
      <li class="list-group-item">
        🎁 QK nhận được quà <b>cj1jd</b>
        <button class="btn btn-warning btn-sm">Đổi quà</button>
      </li>
      <li
        v-for="coupon in couponArray"
        :key="coupon._id"
        :coupon="coupon"
        class="list-group-item"
      >
        🎁 QK nhận được mã coupon <b>{{ coupon }}</b>
        <button class="btn btn-secondary btn-sm">Copy mã</button>
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
export default {
  name: "ListCoupon",
  props: ["couponArray"],
  setup() {
    return {};
  },
};
</script>

<style>
#list-coupon {
  position: fixed;
  right: 20px;
  bottom: 20px;
}
button {
  margin-left: 8px;
}
</style>
