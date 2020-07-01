<template>
  <div class="container">
    <div v-for="(item, idx) in specList" :key="idx">
      <p class="title">{{ item.title }}</p>
      <div class="specBox">
        <span
          v-for="(item2, idx2) in item.list"
          :key="idx2"
          :class="{
            specOption: true,
            specAction: isActive(item2),
            specDisabled: !isOption(item2)
          }"
          @click="clickOption(isOption(item2), item2, idx)"
          >{{ item2 }}</span
        >
      </div>
    </div>
    <div :class="['btn', { active: canBuy }]" @click="clickBuy">购买</div>
    <div class="btn" @click="loadData">重载数据</div>
    <div v-show="loading" class="loading">加载中...</div>
  </div>
</template>

<script>
import { options, combinations } from "./source";

export default {
  data() {
    return {
      timer: null,
      loading: false,
      specList: [], // 规格列表
      specCombinationList: [], // 可选的规格组合
      specsS: [], // 已选的规格
      adjoinArray: [] // 矩阵
    };
  },
  computed: {
    // 可选项数组
    optionSpecs() {
      return this.getSpecscOptions(this.specsS);
    },
    // 是否可以购买
    canBuy() {
      return this.specsS.every(Boolean);
    },
    // 顶点数组
    vertex() {
      return this.specList.reduce(
        (total, current) => [...total, ...current.list],
        []
      );
    },
    // 矩阵长度
    len() {
      return this.vertex.length;
    }
  },
  created() {
    this.loadData();
  },
  beforeDestroy() {
    clearTimeout(this.timer);
  },
  methods: {
    // 模拟请求接口
    loadData() {
      this.loading = true;
      clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        this.loading = false;
        this.specList = options;
        this.specCombinationList = combinations;
        this.init();
      }, 1000);
    },
    // 当前规格是否可选
    isOption(val) {
      return this.optionSpecs.includes(val);
    },
    // 当前规格是否被选
    isActive(val) {
      return this.specsS.includes(val);
    },
    // 初始化
    init() {
      // 以下两行代码除了可以初始化，还能清空数据，因为每次请求接口后数据都有可能不同
      // 矩阵数组
      this.adjoinArray = Array(this.len * this.len).fill(0);
      // 已选的规格数组
      this.specsS = Array(this.specList.length).fill("");
      this.initSpec();
      this.initSameLevel();
    },
    // 根据可选规格组合填写邻接矩阵的值
    initSpec() {
      this.specCombinationList.forEach(item => this.fillInSpec(item.specs));
    },
    // 在可选规格组合内同级顶点填写1
    initSameLevel() {
      this.specList.forEach(item => {
        const params = [];
        // 获取同级别顶点
        item.list.forEach(val => {
          if (this.optionSpecs.includes(val)) params.push(val);
        });
        // 同级点位创建
        this.fillInSpec(params);
      });
    },
    // 填写邻接矩阵的值
    fillInSpec(params) {
      params.forEach(item => this.setAdjoinVertexs(item, params));
    },
    // 传入一个顶点，和当前顶点可达的顶点数组，将对应位置置为1
    setAdjoinVertexs(id, sides) {
      const pIndex = this.vertex.indexOf(id);
      sides.forEach(item => {
        const index = this.vertex.indexOf(item);
        this.adjoinArray[pIndex * this.len + index] = 1;
      });
    },
    // 传入顶点的值，获取该顶点的列
    getVertexCol(id) {
      const idx = this.vertex.indexOf(id) * this.len;
      return this.adjoinArray.slice(idx, idx + this.len);
    },
    // 传入一个顶点数组，求出该数组所有顶点的列的和（抽取求交集和合集的功能）
    getColSum(params) {
      const paramsVertex = params.map(id => this.getVertexCol(id));
      const paramsVertexSum = [];
      this.vertex.forEach((item, index) =>
        paramsVertexSum.push(
          paramsVertex
            .map(val => val[index])
            .reduce((total, current) => (total += current))
        )
      );
      return paramsVertexSum;
    },
    // 传入一个顶点数组，求出交集或并集，默认求并集，isUnions等于true就求交集
    getIntersectionOrUnion(params, isUnions = false) {
      const paramsColSum = this.getColSum(params);
      const arr = [];
      paramsColSum.forEach(
        (item, idx) =>
          (isUnions ? item >= params.length : item) &&
          arr.push(this.vertex[idx])
      );
      return arr;
    },
    // 传入已选的规格数组，查询出可选规格
    getSpecscOptions(params) {
      // 判断已选的规格specsS数组是否每项都是空字符串
      // 已经选了规格，过滤一下可选项
      if (params.some(Boolean))
        // 过滤，把已选的规格传进去
        return this.getIntersectionOrUnion(params.filter(Boolean), true);
      // 没有选任何规格，返回所有可选项
      else return this.getIntersectionOrUnion(this.vertex);
    },
    // 点击选项
    clickOption(bool, text, idx) {
      // 排除可选规格里面没有的规格
      if (this.specsS[idx] !== text && !bool) return;
      // 根据text判断是否已经被选中了，如果已选则取消选中
      this.specsS[idx] = this.specsS[idx] === text ? "" : text;
      // vue2的原理导致，要使用数组的方法才能更新视图层，也可以用$set
      this.specsS = this.specsS.slice();
    },
    // 点击购买
    clickBuy() {
      if (this.canBuy) alert("购买成功");
      else alert("请先选择完所有规格");
    }
  }
};
</script>

<style>
.container {
  width: 400px;
  position: absolute;
  margin: auto;
  left: 0;
  right: 0;
  top: 50px;
}
.title {
  font-size: 16px;
  line-height: 24px;
  color: #262626;
}
.specBox {
  margin: 5px 0 5px 0;
}
.specOption {
  margin-left: 20px;
  background-color: #f3f3f3;
  padding: 5px 10px 5px 10px;
  color: #505257;
  border: 1px solid #f3f3f3;
}
.specAction {
  background-color: #fef6f4;
  color: #e34a40;
  border: 1px solid #e34a40;
}
.specDisabled {
  color: #bebebe;
}
.btn {
  float: right;
  margin-top: 20px;
  margin-left: 20px;
  line-height: 30px;
  padding: 0 20px;
  background: #999;
  color: white;
}
.active {
  background: red;
}
.loading {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: white;
}
</style>
