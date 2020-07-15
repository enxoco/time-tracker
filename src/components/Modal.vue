<template>
  <div class="modal" :class="{ active : active }">
    <div class="modal-heading">
      <h2 class="modal-title">Are you sure you want to delete this task?</h2>
    </div>
    <div class="modal-controls">
      <button @click="$emit('confirm-delete', pendingDelete)" class="confirm">Delete</button>
      <button @click="$emit('close-modal')" class="cancel">Cancel</button>
    </div>
  </div>
</template>

<style scoped>
.modal {
  z-index: 99999;
  padding: 50px;
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  right: 0;
  background: #BFD200;
  display: none;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #f1faee;
  border-radius: 5px;
  border: 1px solid #007F5F;
box-shadow: 10px 17px 31px 0px #007F5F;
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
}
.modal button {
    border-radius: 10px;
    padding: 20px 10px;
    margin-left:10px;
    margin-right: 10px;
    width: 100px;
    box-shadow: none;
    margin-top: 20px;
    border:1px solid #007F5F;
}
.modal button.confirm {
    background: #EEEF20;
    color: #007F5F;
}
.modal button.cancel {
    background: #2B9348;
    color: #EEE5E9;
    }
.modal.active {
  display: flex;
}
.modal-title {
  text-align: center;
  color: #007F5F;
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
}
</style>
<script>
export default {
  props: ["active", "pendingDelete"]
};
</script>