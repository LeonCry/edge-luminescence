<template>
  <section class="edge-grid h-[1200px] w-[1800px] bg-zinc-200 p-10" :style="{ gridTemplateAreas }">
    <div v-for="n in gridAreaNames" :key="n" :style="{ gridArea: n }">{{ n }}</div>
  </section>
</template>
<script setup lang="ts">
import { random } from 'radash';
interface blockType {
  width: number;
  height: number;
}
// 图片长宽比 = 3:2
const ratio = 2 / 3;
//第一次请求
const t = 1;
// 一次出10个
const count = 10;
// 最大允许 1个3x3 + 1个3x2 + 1个3x1 共 18个 块宽度
const wTList = [3, 6, 9];
const hTList = wTList.map((w) => w * ratio);
const continueRow = ref(0);
const maxWidth = wTList.reduce((p, c) => p + c, 0);
const gridAreaNames = ref<string[]>(Array.from({ length: count }, (_, i) => `ar_${t}_${i}`));
const generatedArea = [Array.from({ length: maxWidth }, () => '.')];
//获取一行中的连续空白长度 [startColumn, endColumn][];
const getEmptyWidth = async (row: number) => {
  if (row >= generatedArea.length) {
    generatedArea.push(Array.from({ length: maxWidth }, () => '.'));
    return [[0, maxWidth - 1]] as [number, number][];
  }
  const ew: [number, number][] = [];
  let ring = false;
  generatedArea[row].forEach((item, index) => {
    if (!ring && item !== '.') {
      ring = false;
    } else if (!ring && item === '.') {
      ew.push([index, 0]);
      ring = true;
    } else if (ring && item !== '.') {
      ew[ew.length - 1][1] = index - 1;
      ring = false;
    } else if (ring && index === maxWidth - 1) {
      ew[ew.length - 1][1] = index;
    }
  });
  await new Promise((resolve) => setTimeout(resolve));
  return ew;
};
//根据空白长度生成块类型
const fillingByEmptyWidth = (emptyWidth: [number, number]) => {
  let width = emptyWidth[1] - emptyWidth[0] + 1;
  const blocks: blockType[] = [];
  while (width) {
    const maxR = wTList.findLastIndex((w) => w <= width);
    const r = random(0, maxR);
    width -= wTList[r];
    blocks.push({ width: wTList[r], height: hTList[r] });
  }
  return blocks;
};
//根据块类型生成grid模板区域
const getGridTemplateArea = (
  row: number,
  col: [number, number],
  blocks: blockType[],
  names: string[],
) => {
  let startCol = col[0];
  blocks.forEach((bk) => {
    const name = names.shift();
    if (!name) return;
    for (let r = row; r < row + bk.height; r++) {
      for (let c = startCol; c < startCol + bk.width; c++) {
        if (!generatedArea[r]) {
          generatedArea.push(Array.from({ length: maxWidth }, () => '.'));
        }
        generatedArea[r][c] = name;
      }
    }
    startCol += bk.width;
  });
};
//生成函数
const handleGenerating = async () => {
  let row = continueRow.value;
  const curGeneratedName = [...gridAreaNames.value];
  while (curGeneratedName.length) {
    const emptyWidth = await getEmptyWidth(row);
    if (!emptyWidth.length) {
      row++;
      continue;
    }
    emptyWidth.forEach((col) => {
      const blocks = fillingByEmptyWidth(col);
      getGridTemplateArea(row, col, blocks, curGeneratedName);
    });
  }
  continueRow.value = row;
  await new Promise((resolve) => setTimeout(resolve));
};
const getGridTemplateAreas = async () => {
  await handleGenerating();
  const res = generatedArea
    .map((row) => row.join(' '))
    .map((r) => "'" + r + "'")
    .join(' ');
  gridTemplateAreas.value = res;
};
const gridTemplateAreas = ref();
getGridTemplateAreas();
</script>
<style scoped lang="scss">
.edge-grid {
  display: grid;
  gap: 4px;
  grid-template-rows: repeat(auto-fill, 1fr);
  grid-auto-flow: row dense;
}
.edge-grid > * {
  border: 1px solid #000;
  background-color: lightgray;
  padding: 4px;
}
</style>
