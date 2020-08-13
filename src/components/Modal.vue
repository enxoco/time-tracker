<template>
  <div class="modal" :class="{ active : active }">
    <div class="modal-heading">
      <h2 class="modal-title">Delete Task?</h2>
      <p>Are you sure you want to delete this task?</p>
      <span><small>This action cannot be undone</small></span>
    </div>
    <div class="modal-controls">
      <a @click="$emit('confirm-delete', pendingDelete)" class="confirm">Confirm Deletion</a>
      <a @click="$emit('close-modal')" class="cancel">Cancel Deletion</a>
    </div>
<label class="container">Don’t show this confirmation in the future
  <input type="checkbox" @click="$emit('toggle-modal')">
  <span class="checkmark"></span>
</label>

  </div>
</template>

<style scoped>
* {
  font-family: 'Oxygen';
}
.modal {
  z-index: 99999;
  padding: 50px;
  position: absolute;
  left: 33%;
  top: 25%;
  bottom: 25%;
  right: 33%;
  background: #222831;
  display: none;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  color: #f1faee;
  border-radius: 5px;
  min-height: 400px;


}
.modal a {
    border-radius: 10px;
    padding: 20px 10px;
    margin-left:10px;
    margin-right: 10px;
    margin-top: 20px;
    font-size: 18px;
    font-weight: 700;
    cursor: pointer;
}
.modal a:hover {
  background: #eee;
  color: #222831;
}
.modal .confirm {
    background: #FB5252;
    color: #eeeeee;
}
.modal .cancel {
    background: #32E0C4;
    color: #eeeeee;
    }
.modal.active {
  display: flex;
}
.modal-title {
  text-align: center;
  color: #EEEEEE;
  font-size: 72px;

}
p {
  font-size: 24px;
}
/* The container */
.container {
  display: block;
  position: relative;
  padding-left: 35px;
  margin-bottom: 12px;
  cursor: pointer;
  font-size: 22px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

/* Hide the browser's default checkbox */
.container input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

/* Create a custom checkbox */
.checkmark {
  position: absolute;
  top: 0;
  left: 0;
  height: 25px;
  width: 25px;
  background-color: #eee;
}

/* On mouse-over, add a grey background color */
.container:hover input ~ .checkmark {
  background-color: #ccc;
}

/* When the checkbox is checked, add a blue background */
.container input:checked ~ .checkmark {
  background-color: #2196F3;
}

/* Create the checkmark/indicator (hidden when not checked) */
.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

/* Show the checkmark when checked */
.container input:checked ~ .checkmark:after {
  display: block;
}

/* Style the checkmark/indicator */
.container .checkmark:after {
  left: 9px;
  top: 5px;
  width: 5px;
  height: 10px;
  border: solid white;
  border-width: 0 3px 3px 0;
  -webkit-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  transform: rotate(45deg);
}
@media (max-width: 425px) {
  .modal {
    top: 0;
    right: 0;
    left: 0;
    bottom: 0;
    padding: 50px 25px;
  }
  .modal a {
    font-size: 15px;
  }
  .modal-title {
    font-size: 52px;
  }
}
</style>
<script>
export default {
  props: ["active", "pendingDelete"],
    data: function() {
    return {
    };
  },
};
</script>