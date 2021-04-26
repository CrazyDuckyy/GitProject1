<template>
  <div class="header">
    <div class="balance">
      <div class="head">
        <p>可用余额(元)</p>
        <button>申请提现</button>
      </div>
      <hr />
      <div class="main">20.00</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableData: [
        {
          yue: "20.00",
        },
      ],
    };
  },
};
</script>

<style  scoped>
.header {
  margin-left: 20px;
}
.balance {
  width: 23%;
  /* font-size: 30px; */
  border: 2px solid red;
}

.head {
  font-size: 20px;
  height: 50px;
  display: flex;
  justify-content: space-around;
}
.main{
    font-size: 30px;
    height: 70px;
}
</style>
