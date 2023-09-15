<template>
  <div class="hello">
    <h6><input class="date" type="text" v-model="year">年<input class="date" type="text" v-model="month">月对账单</h6>
    <div class="box" v-for="item,i in box" :key="item">
      <p class="box-title">
        <input type="text" v-model="item.name">
        <button @click="delCardItem(i)">删除</button>
      </p>
      <div class="box-item" v-for="subItem in item.items" :key="subItem">
        <input type="text" v-model="subItem.name" :readonly="subItem.type === 'diff'">：
        <input type="text" v-model="subItem.value" :readonly="subItem.type === 'diff'">
        <select v-model="subItem.type">
          <option value="in">收入</option>
          <option value="out">支出</option>
          <option value="total">余额</option>
          <option value="diff">差值</option>
        </select>
        <label>不计入：<input type="checkbox" v-model="subItem.ignore" :readonly="subItem.type === 'diff'" :true-value="true" :false-value="false"></label>
      </div>
    </div>
    <div class="btn-wrap">
      <button @click="addCard">添加银行卡</button>
    </div>
    <div class="items" v-for="item,i in countItem" :key="item">
      <input type="text" v-model="item.name" :readonly="item.type === 'over'">：
      <input type="text" v-model="item.value" :readonly="item.type === 'over'">
      <select v-model="item.type">
        <option value="over">其他支出</option>
        <option value="out">支出</option>
      </select>
      <button @click="delExPenseItem(i)">删除</button>
    </div>
    <div class="btn-wrap">
      <button @click="addExpense">添加支出项</button>
    </div>
    <div class="items">总收入： {{totalIncome}}</div>
    <div class="items">总支出： {{totalExpenditure}}</div>
    <div class="items">当月结余： {{currentMonthBalance}}</div>
    <div class="items">总余额： {{totalBalance}}</div>
    <div class="btn-wrap">
      <button @click="exportText">导出</button>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watchEffect, computed } from "vue";

const year = ref('2023')
const month = ref('01')

const box = reactive([
  {
    name: '兴业银行',
    items: [
      { name: '收入', value: 0, type: 'in', ignore: false },
      { name: '支出', value: 0, type: 'out', ignore: false },
      { name: '余额', value: 0, type: 'total', ignore: false }
    ]
  },
  {
    name: '招商银行',
    items: [
      { name: '收入', value: 0, type: 'in', ignore: false },
      { name: '支出', value: 0, type: 'out', ignore: false },
      { name: '余额', value: 0, type: 'total', ignore: false }
    ]
  },
  {
    name: '支付宝',
    items: [
      { name: '收入', value: 0, type: 'in', ignore: true },
      { name: '支出', value: 0, type: 'out', ignore: true },
      { name: '实际支出', value: 0, type: 'diff', ignore: true }
    ]
  },
  {
    name: '微信',
    items: [
      { name: '收入', value: 0, type: 'in', ignore: true },
      { name: '支出', value: 0, type: 'out', ignore: true },
      { name: '实际支出', value: 0, type: 'diff', ignore: true }
    ]
  }
])

const countItem = reactive([
  { name: '生活其他', value: 0, type: 'over' },
  { name: '转账王琨', value: 0, type: 'out' },
])

// 总收入
const totalIncome = computed(() => {
  return box.reduce((sum, item) => {
    if (Array.isArray(item.items)) {
      sum += item.items.reduce((subSum, subItem) => {
        if (subItem.type === 'in' && !subItem.ignore) {
          subSum += Number(subItem.value)
        }
        return subSum
      }, 0)
    }
    return sum
  }, 0)
})
// 总支出
const totalExpenditure = computed(() => {
  return box.reduce((sum, item) => {
    if (Array.isArray(item.items)) {
      sum += item.items.reduce((subSum, subItem) => {
        if (subItem.type === 'out' && !subItem.ignore) {
          subSum += Number(subItem.value)
        }
        return subSum
      }, 0)
    }
    return sum
  }, 0)
})
// 当月结余
const currentMonthBalance = computed(() => totalIncome.value - totalExpenditure.value)
// 总余额
const totalBalance = computed(() => {
  return box.reduce((sum, item) => {
    if (Array.isArray(item.items)) {
      sum += item.items.reduce((subSum, subItem) => {
        if (subItem.type === 'total' && !subItem.ignore) {
          subSum += Number(subItem.value)
        }
        return subSum
      }, 0)
    }
    return sum
  }, 0)
})

watchEffect(() => {
  box.forEach(item => {
    if (Array.isArray(item.items)) {
      let inCount = 0
      let outCount = 0

      item.items.forEach(subItem => {
        if (subItem.type === 'in') {
          inCount += Number(subItem.value)
        } else if (subItem.type === 'out') {
          outCount += Number(subItem.value)
        } else if (subItem.type === 'total') {
          subItem.value = inCount - outCount
        } else if (subItem.type === 'diff') {
          subItem.value = outCount - inCount
        }
      })
    }
  })
})

watchEffect(() => {
  let total = ref(totalExpenditure.value)
  countItem.forEach(item => {
    if (item.type === 'out') {
      total.value -= item.value
    } else if (item.type === 'over') {
      item.value = total
    }
  })
})

// 添加银行卡
const addCard = () => {
  box.push({
    name: '',
    items: [
      { name: '收入', value: 0, type: 'in', ignore: false },
      { name: '支出', value: 0, type: 'out', ignore: false },
      { name: '余额', value: 0, type: 'total', ignore: false }
    ]
  })
}
// 添加支出项
const addExpense = () => {
  countItem.push({ name: '', value: 0, type: 'out' })
}
const delCardItem = (i) => {
  box.splice(i, 1)
}
const delExPenseItem = (i) => {
  countItem.splice(i, 1)
}

// 导出
const exportText = async () => {
  const text = `${year.value}年${month.value}月对账单
${box.map(item => ` 
${item.name}：
${item.items.map(subItem => `     ${subItem.name}： ${Number(subItem.value).toFixed(2)}`).join('\n')}
 `).join('')} 
${countItem.map(item => `${item.name}： ${Number(item.value).toFixed(2)}`).join('\n')}
 
总收入： ${Number(totalIncome.value).toFixed(2)}
总支出： ${Number(totalExpenditure.value).toFixed(2)}
当月结余： ${Number(currentMonthBalance.value).toFixed(2)}
总余额： ${Number(totalBalance.value).toFixed(2)}`
  console.log(text)
  await navigator.clipboard.writeText(text)
  alert('已复制到剪贴板！')
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scoped>
.hello {
  width: 450px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-size: 12px;
  color: #999;

  input[type="text"],
  select {
    width: 100px;
    padding: 5px 10px;
    border: 1px solid #aaa;
    border-radius: 5px;
    margin-right: 5px;
  }
  label {
    display: inline-flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
  }
  button {
    background: none;
    outline: 0 none;
    padding: 5px 10px;
    border: 1px solid #aaa;
    border-radius: 5px;
  }
  .date {
    width: 50px;
    text-align: center;
  }
  .box {
    width: 100%;
    padding-bottom: 10px;
    margin-bottom: 10px;
    border-bottom: 1px solid #aaa;
    .box-title {
      margin-bottom: 10px;
      text-align: left;
    }
    .box-item {
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: flex-start;
      padding: 5px 0;
    }
  }
  .items {
    width: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: flex-start;
    padding: 5px 0;
  }
  .btn-wrap {
    width: 100%;
    text-align: right;
    padding-bottom: 10px;
    margin-bottom: 10px;
    border-bottom: 1px solid #aaa;
  }
}
</style>
