<template>
  <div class="goods-sku">
    <dl v-for="item in goods.specs" :key="item.id">
      <dt>{{ item.name }}</dt>
      <dd>
        <template v-for="val in item.values" :key="val.name">
          <img
            @click="changeSku(item, val)"
            v-if="val.picture"
            :class="{ selected: val.selected,disabled: val.disabled}"
            :src="val.picture"
            :title="val.name"
            alt=""
          />
          <span
            @click="changeSku(item, val)"
            v-else
            :class="{ selected: val.selected, disabled: val.disabled }"
            >{{ val.name }}</span
          >
        </template>
      </dd>
    </dl>
  </div>
</template>
<script>
import powerSet from '@/vander/power-set'
// 得到一个路径字典对象
const getPathMap = (skus) => {
  // 1、得到所有sku集合 props.goods.skus
  // 2、从所有的sku筛选有效的
  // 3、根据有效的sku使用power-set算法得到集合
  // 4、根据自己路径字典对象中存储 key -  value

  const pathMap = {}

  skus.forEach((item) => {
    if (item.inventory > 0) {
      // 计算子级 当前有库存的sku
      // 例如：[1,2,3] => [[1],[2],[3],[1,2],[1,3],[2,3],[1,2,3]]

      // 可选值数组  简单数据
      const valueArr = item.specs.map((val) => val.valueName)
      // console.log(valueArr)

      // 可选值数组，子集
      const valueArrPowerset = powerSet(valueArr)
      // console.log(valueArrPowerset)

      // 遍历子级 插入数据
      valueArrPowerset.forEach((arr) => {
        // 根据arr得到字符串 key 约定key是:['蓝色','中国'] ===> '蓝色-中国'
        const key = arr.join('-')
        // console.log(key)

        // 给pathMap设置数据
        if (pathMap[key]) {
          pathMap[key].push(1)
        } else {
          pathMap[key] = [item.id]
          // obj[name] = [id]
        }
      })
    }
  })
  return pathMap
}

// 更新按钮禁用状态
const updateDisabledState = (specs, pathMap) => {
  // 1.约定每一个按钮的状态 由本身的disable数据来控制
  specs.forEach((item, i) => {
    const selectedValues = getSelectedValues(specs) // 单击蓝色：['蓝色',undefind,undefind]
    item.values.forEach((val) => {
      // 2.判断是否选中 选中就不用选中
      if (val.selected) return // 排除当前选中的
      // 3.如果没有选中，用当前的值按照顺序套入循环中的值数据
      selectedValues[i] = val.name // 此处索引为1，selectedValues中新值数据：黑色
      // 4.剔除undefinde数据，按照分隔符剔除key
      const key = selectedValues.filter((value) => value).join('-')
      // console.log('正在使用：', key)
      // 5.按照key 去路径字典中，查找是否有数据 有可以点击，有就禁用（true）
      val.disable = !pathMap[key] // 字典中，无法查询 '蓝色-黑色' 返回false 取反就表示禁用
    })
  })
}

// 根据每一项生成 查询数组
const getSelectedValues = (specs) => {
  const arr = []
  specs.forEach((item) => {
    // 选中的按钮对象
    const selectedVal = item.values.find((val) => val.selected)
    // 选中的按钮对象名字
    arr.push(selectedVal ? selectedVal.name : undefined)
  })
  return arr
}
export default {
  name: 'GoodsSku',
  props: {
    goods: {
      type: Object,
      default: () => ({})
    }
  },
  setup (props) {
    const pathMap = getPathMap(props.goods.skus) // 获取字典信息
    // console.log(pathMap)

    // 组件初始化，更新按钮的状态
    updateDisabledState(props.goods.specs, pathMap)

    // 选中与取消选中 约定在每一个按钮都拥有选中状态数据 ：selected
    // 1、点击的是已选中的，则需要取消当前选中
    // 2、点击的是未选中的，消除选中的，自己再选中
    const changeSku = (item, val) => {
      // 当按钮是禁用的，阻止程序运行
      if (val.disable) return
      if (val.selected) {
        val.selected = false
      } else {
        // console.log('item:', item)
        item.values.forEach((obj) => {
          obj.selected = false
        })
        val.selected = true
      }
      // 点击按钮时，更新按钮状态
      // getSelectedValues(props.goods.specs)
      updateDisabledState(props.goods.specs, pathMap)
    }

    return { changeSku }
  }
}
</script>
<style scoped lang="less">
.sku-state-mixin () {
  border: 1px solid #e4e4e4;
  margin-right: 10px;
  cursor: pointer;
  &.selected {
    border-color: @xtxColor;
  }
  &.disabled {
    opacity: 0.6;
    border-style: dashed;
    cursor: not-allowed;
  }
}
.goods-sku {
  padding-left: 10px;
  padding-top: 20px;
  dl {
    display: flex;
    padding-bottom: 20px;
    align-items: center;
    dt {
      width: 50px;
      color: #999;
    }
    dd {
      flex: 1;
      color: #666;
      > img {
        width: 50px;
        height: 50px;
        .sku-state-mixin ();
      }
      > span {
        display: inline-block;
        height: 30px;
        line-height: 28px;
        padding: 0 20px;
        .sku-state-mixin ();
      }
    }
  }
}
</style>
