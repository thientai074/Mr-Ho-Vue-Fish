<template>
  <div id="myModal" class="modal fade" role="dialog">
    <div class="modal-dialog">
      <!-- Modal content-->
      <div class="modal-content">
        <div class="modal-body">
          <div id="msg-gift" class="text-center py-3">
            <h4 class="modal-title mb-2">Chúc mừng Quý khách đã nhận được</h4>
            <h3 class="text-danger mb-3">
              coupon giảm {{ couponRN }}% or quà {{ couponR }}
            </h3>
          </div>
          <div class="action text-center mb-3">
            <button class="btn btn-default">Copy coupon</button>
            <button class="btn btn-primary">Chia sẻ lên facebook</button>
          </div>
          <div class="alert alert-warning mb-0">
            <p class="m-0">
              Coupon có giá trị sử dụng đến hết ngày <b>31/08/2022</b>
            </p>
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-warning" data-dismiss="modal">
            Tiếp tục
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  name: "GiftModal",
  props: ["couponR", "couponRN"],
  setup() {
    return {};
  },
};
</script>

<style></style>
