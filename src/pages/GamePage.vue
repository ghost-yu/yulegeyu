<template>
  <div id="gamePage">
    <a-row align="space-between">
      <a-button style="margin-bottom: 8px" @click="doBack"> 返回</a-button>
      <a-button>块数：{{ clearBlockNum }} / {{ totalBlockNum }}</a-button>
    </a-row>
    <!-- 胜利 -->
    <a-row align="center">
      <div v-if="gameStatus === 3" style="text-align: center">
        <h2>恭喜，你赢啦！🎉</h2>
        <img alt="程序员鱼皮" src="../assets/kunkun.png" />
        <my-ad style="margin-top: 16px" />
      </div>
    </a-row>
    <!-- 分层选块 -->
    <a-row align="center">
      <div v-show="gameStatus > 0" class="level-board">
        <div v-for="(block, idx) in levelBlocksVal" :key="idx">
          <div
            v-if="block.status === 0"
            class="block level-block"
            :class="{
              disabled: !isHolyLight && block.lowerThanBlocks.length > 0,
            }"
            :data-id="block.id"
            :style="{
              zIndex: 100 + block.level,
              left: block.x * widthUnit + 'px',
              top: block.y * heightUnit + 'px',
            }"
            @click="() => doClickBlock(block)"
          >
            <template v-if="getImage(block.type)">
              <img :src="getImage(block.type)" class="block-img" />
            </template>
            <template v-else>
              {{ block.type }}
            </template>
          </div>
        </div>
      </div>
    </a-row>
    <!-- 随机选块 -->
    <a-row align="space-between" class="random-board">
      <div
        v-for="(randomBlock, index) in randomBlocksVal"
        :key="index"
        class="random-area"
      >
        <div
          v-if="randomBlock.length > 0"
          :data-id="randomBlock[0].id"
          class="block"
          @click="() => doClickBlock(randomBlock[0], index)"
        >
          <template v-if="getImage(randomBlock[0].type)">
            <img :src="getImage(randomBlock[0].type)" class="block-img" />
          </template>
          <template v-else>
            {{ randomBlock[0].type }}
          </template>
        </div>
        <!-- 隐藏 -->
        <div
          v-for="num in Math.max(randomBlock.length - 1, 0)"
          :key="num"
          class="block disabled"
        >
          <span v-if="canSeeRandom">
            {{ randomBlock[num].type }}
          </span>
        </div>
      </div>
    </a-row>
    <!-- 槽位 -->
    <a-row v-if="slotAreaVal.length > 0" align="center" class="slot-board">
      <div v-for="(slotBlock, index) in slotAreaVal" :key="index" class="block">
        <template v-if="slotBlock && getImage(slotBlock.type)">
          <img :src="getImage(slotBlock.type)" class="block-img" />
        </template>
        <template v-else>
          {{ slotBlock?.type }}
        </template>
      </div>
    </a-row>
    <!-- 技能 -->
    <div class="skill-board">
      <a-space>
        <a-button size="small" @click="doRevert">撤回</a-button>
        <a-button size="small" @click="doRemove">移出</a-button>
        <a-button size="small" @click="doShuffle">洗牌</a-button>
        <a-button size="small" @click="doBroke">破坏</a-button>
        <a-button size="small" @click="doHolyLight">圣光</a-button>
        <a-button size="small" @click="doSeeRandom">透视</a-button>
      </a-space>
    </div>
  </div>
</template>

<script setup lang="ts">
import useGame from "../core/game";
import { onMounted } from "vue";
import { useRouter } from "vue-router";
import MyAd from "../components/MyAd.vue";
import { useGlobalStore } from "../core/globalStore";

const router = useRouter();

const {
  gameStatus,
  levelBlocksVal,
  randomBlocksVal,
  slotAreaVal,
  widthUnit,
  heightUnit,
  totalBlockNum,
  clearBlockNum,
  isHolyLight,
  canSeeRandom,
  doClickBlock,
  doStart,
  doShuffle,
  doBroke,
  doRemove,
  doRevert,
  doHolyLight,
  doSeeRandom,
} = useGame();

// global store to read current gameConfig (animals order etc)
const { gameConfig } = useGlobalStore();

// 动态导入 `src/assets/animals` 下的所有图片（如果你把素材放到该目录下）
// 使用 Vite 的 import.meta.glob，当目录为空时返回空对象。
const imageModules = import.meta.glob("../assets/animals/*", {
  eager: true,
  as: "url",
}) as Record<string, string>;
const imageUrls = Object.values(imageModules || {});

/**
 * 根据方块的 type 返回图片 URL（优先），没有图片则返回 null
 */
const getImage = (type: string) => {
  if (!imageUrls || imageUrls.length === 0) return null;
  // 尝试按 gameConfig.animals 的顺序做映射
  const idx = gameConfig.animals.indexOf(type);
  const useIdx = idx >= 0 ? idx % imageUrls.length : 0;
  return imageUrls[useIdx];
};

/**
 * 回上一页
 */
const doBack = () => {
  router.back();
};

onMounted(() => {
  doStart();
});
</script>

<style scoped>
.level-board {
  position: relative;
}

.level-block {
  position: absolute;
}

.random-board {
  margin-top: 8px;
}

.random-area {
  margin-top: 8px;
}

.slot-board {
  border: 10px solid saddlebrown;
  margin: 16px auto;
  width: fit-content;
}

.skill-board {
  text-align: center;
}

.block {
  font-size: 28px;
  width: 42px;
  height: 42px;
  line-height: 42px;
  border: 1px solid #eee;
  background: white;
  text-align: center;
  vertical-align: top;
  display: inline-block;
}

.block-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: inline-block;
}

.disabled {
  background: grey;
  cursor: not-allowed;
}
</style>
