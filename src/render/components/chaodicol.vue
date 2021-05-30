<template>
  <el-container>
    <el-aside width="250px" style="height: 100vh; padding: 20px">
      <div class="title">投入仓位（本位币）</div>
      <el-input-number
        v-model="put_position"
        :precision="7"
        :step="0.1"
        :min="0.0000001"
      ></el-input-number>
      <div class="title">分仓网格数</div>
      <el-input-number
        v-model="grid_num"
        :precision="0"
        :step="1"
        :min="1"
      ></el-input-number>
      <div class="title">仓位等比乘数</div>
      <el-input-number
        v-model="grid_multiplier"
        :precision="2"
        :step="0.1"
        :min="0.1"
      ></el-input-number>
      <div class="title">最高区间（现价）</div>
      <el-input-number
        v-model="max_price"
        :precision="7"
        :step="0.1"
        :min="0.0000001"
      ></el-input-number>
      <div class="title">最低区间（预测位置）</div>
      <el-input-number
        v-model="min_price"
        :precision="7"
        :step="0.1"
        :min="0.0000001"
      ></el-input-number>
      <el-button class="margin-top" type="primary" @click="calcPosition()"
        >生成网格</el-button
      >
      <div class="title">
        版本号：0.0.1
      </div>
    </el-aside>
    <el-main>
      <el-row style="height: 100vh; padding: 20px">
        <el-table :data="result" style="width: 100%">
          <el-table-column prop="pos_index" label="仓位序号" width="100">
          </el-table-column>
          <el-table-column prop="per_position" label="仓位占比%" width="100">
          </el-table-column>
          <el-table-column
            prop="position"
            label="合计仓位(本位币）"
            width="180"
          >
          </el-table-column>
          <el-table-column
            prop="buy_price"
            label="买入价格(交易币）"
            width="180"
          >
          </el-table-column>
          <el-table-column prop="count" label="数量(交易币）">
          </el-table-column>
        </el-table>
      </el-row>
    </el-main>
  </el-container>
</template>

<script setup>
import { ref } from "vue";

// const state = reactive({ count: 0 })
let grid_num = ref(10);
let grid_multiplier = ref(1);
let put_position = ref(1);
let max_price = ref(1);
let min_price = ref(1);

let result = ref([]);

function calcPosition() {
  result.value = [];
  // 计算等比的比例
  let per_list = [1];

  for (let i = 0; i < grid_num.value - 1; i++) {
    let temp_num = per_list[i] * grid_multiplier.value;
    per_list.push(temp_num);
  }
  // console.log(per_list);

  // 转换成仓位百分比
  let per_list_per = [];
  let sum_per_list = 0;
  for (let i = 0; i < per_list.length; i++) {
    sum_per_list = sum_per_list + per_list[i];
  }
  for (let i = 0; i < per_list.length; i++) {
    let temp_num = per_list[i] / sum_per_list;
    per_list_per.push(temp_num);
  }
  // console.log(per_list_per);

  // 生成买入仓位列表
  let position_list = [];
  for (let i = 0; i < per_list_per.length; i++) {
    let temp_num = put_position.value * per_list_per[i];
    position_list.push(temp_num);
  }
  // console.log(position_list);

  // 计算买入价格列表
  let price_multiplier = Math.pow(
    max_price.value / min_price.value,
    1 / (grid_num.value - 1)
  );

  // 生成买入价格列表
  let price_list = [];
  for (let i = 0; i < grid_num.value; i++) {
    let temp_num = min_price.value * Math.pow(price_multiplier, i);
    price_list.unshift(temp_num);
  }
  // console.log(price_list);

  // 生成买入数量列表
  let buy_number_list = [];
  for (let i = 0; i < grid_num.value; i++) {
    let temp_num = position_list[i] / price_list[i];
    buy_number_list.push(temp_num);
  }
  // console.log(buy_number_list);

  // 生成结果
  for (let i = 0; i < grid_num.value; i++) {
    let item = {
      pos_index: i + 1,
      buy_price: price_list[i].toFixed(7),
      count: buy_number_list[i].toFixed(7),
      position: position_list[i].toFixed(7),
      per_position: (per_list_per[i] * 100).toFixed(2),
      // sell_price: 0,
      // exp_profits: 0,
    };
    // item.sell_price = item.buy_price * price_multiplier;
    // item.exp_profits = (item.sell_price - item.buy_price) * item.count;
    result.value.push(item);
  }
  // console.log(result.value);
}
</script>

<style scoped>
.title {
  font-size: 10px;
  color: dimgray;
  margin-bottom: 5px;
  margin-top: 20px;
}
.margin-top {
  margin-top: 20px;
}
.padding {
  padding: 20px;
}
.el-main {
  padding: 0;
}
</style>
