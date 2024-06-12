<template>
  <div class="check-box">
    <div class="school-box" v-for="schoolItem in schoolList" :key="schoolItem.id">
      <!-- 学校部分 -->
      <div class="school-item" @click.stop="selectSchool(schoolItem.schoolId)">
        <img v-show="!schoolItem.actived" :class="['icon', { 'active-icon': schoolItem.expand }]"
          src="../assets/img/icon_unactive.png" />
        <img v-show="schoolItem.actived" :class="['icon', { 'active-icon': schoolItem.expand }]"
          src="../assets/img/icon_active.png" />
        <text :class="['name', { 'active-name': schoolItem.actived }]">{{ schoolItem.schoolName }}</text>
        <!-- 学校部分=>复选框盒子 -->
        <div class="checked-btn-box" @click.stop="checkSchool(schoolItem.schoolId)">
          <text v-show="!schoolItem.checked" class="unchecked"></text>
          <img v-show="schoolItem.checked" class="checked-btn" src="../assets/img/icon_checked.png" />
        </div>
      </div>

      <!-- 楼栋部分 -->
      <div v-show="schoolItem.expand" class="build-box" v-for="buildItem in schoolItem.list" :key="buildItem.buildId"
        @click.stop="selectBuild(schoolItem.schoolId, buildItem.buildId)">
        <div class="build-item">
          <IconAdress :color="buildItem.actived ? '#1b6aff' : '#222'"></IconAdress>
          <text :class="['name', { 'active-name': buildItem.actived }]">{{ buildItem.buildName }}</text>
          <!-- 楼栋部分=>复选框盒子 -->
          <div class="checked-btn-box" @click.stop="checkBuild(schoolItem.schoolId, buildItem.buildId)">
            <text v-show="!buildItem.checked" class="unchecked"></text>
            <img v-show="buildItem.checked" class="checked-btn" src="../assets/img/icon_checked.png" />
          </div>
        </div>

        <!-- 楼层部分 -->
        <div v-show="buildItem.expand" class="floor-box">
          <div class="floor-item" v-for="floorItem in buildItem.list" :key="floorItem.floorId">
            <text :class="['name', { 'active-name': floorItem.actived }]">{{ floorItem.floorName }}楼</text>
            <!-- 楼层部分=>复选框盒子 -->
            <div class="checked-btn-box" @click.stop="checkFloor(buildItem.buildId, floorItem.floorId)">
              <text v-show="!floorItem.checked" class="unchecked"></text>
              <img v-show="floorItem.checked" class="checked-btn" src="../assets/img/icon_checked.png" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import IconAdress from "./icons/IconAdress.vue";
import { ref, reactive } from "vue";

const props = defineProps({
  list: { type: Array, required: true }
});

const schoolList = reactive([]);
// 初始化数据
function initListData() {
  const mapList = props.list.map((schoolItem) => {
    schoolItem.actived = false; // 新增属性:文字高亮
    schoolItem.expand = false; // 新增属性:展开子级
    schoolItem.checked = false; // 新增属性:选中(全选)

    schoolItem.list.map((buildItem) => {
      buildItem.actived = false; // 新增属性:文字高亮
      buildItem.expand = false; // 新增属性:展开子级
      buildItem.checked = false; // 新增属性:选中(全选)

      buildItem.list.map((floorItem) => {
        floorItem.actived = false; // 新增属性:文字高亮
        floorItem.checked = false; // 新增属性:选中(全选)
        return floorItem;
      });
      return buildItem;
    });
    return schoolItem;
  });

  schoolList.splice(0, props.list.length, ...mapList);
}

initListData();

const activeSchoolId = ref('') // 点击选中的学校id
const activeBuildId = ref('') // 点击选中的楼栋id
// 点击展开学校
const selectSchool = (curId) => {
  const curSchoolItem = schoolList.find(e => e.schoolId == curId)
  if (activeSchoolId.value == curId) return curSchoolItem.expand = !curSchoolItem.expand

  curSchoolItem.expand = true
  // 关闭其余学校
  schoolList.forEach(e => { if (e.schoolId != curId) e.expand = false })
  activeSchoolId.value = curId
}

// 点击展开楼栋
const selectBuild = (curSchoolId, curBuildId) => {
  const curSchoolItem = schoolList.find(e => e.schoolId == curSchoolId)
  const curBuildItem = curSchoolItem.list.find(e => e.buildId == curBuildId)
  if (activeBuildId.value == curBuildId) return curBuildItem.expand = !curBuildItem.expand

  curBuildItem.expand = true
  // 关闭其余楼栋
  curSchoolItem.list.forEach(e => { if (e.buildId != curBuildId) e.expand = false })
  activeBuildId.value = curBuildId
}

// 点击复选学校
const checkSchool = (curId) => {
  const curSchoolItem = schoolList.find(e => e.schoolId == curId)
  const check = curSchoolItem.checked

  curSchoolItem.checked = !check
  curSchoolItem.actived = !check

  // 楼栋处理
  curSchoolItem.list.forEach(e => {
    e.actived = !check
    e.checked = !check
    // 楼层处理
    e.list.forEach(ele => {
      ele.actived = !check
      ele.checked = !check
    })
  })
}

// 点击复选楼栋
const checkBuild = (schoolId, buildId) => {
  const curSchoolItem = schoolList.find(e => e.schoolId == schoolId)
  const curBuildItem = curSchoolItem.list.find(e => e.buildId == buildId)
  const check = curBuildItem.checked

  curBuildItem.actived = !check
  curBuildItem.checked = !check

  // 楼层处理
  curBuildItem.list.forEach(e => {
    e.actived = !check
    e.checked = !check
  })

  // 优先-回显判断学校是否高亮
  callBackSchool()
  // 回显判断楼栋是否高亮
  callBackBuild()
}

// 点击复选楼层
const checkFloor = (buildId, floorId) => {
  const curSchoolItem = schoolList.find(e => e.schoolId == activeSchoolId.value)
  const curBuildItem = curSchoolItem.list.find(e => e.buildId == buildId)
  const curFloorItem = curBuildItem.list.find(e => e.floorId == floorId)
  curFloorItem.actived = !curFloorItem.actived
  curFloorItem.checked = !curFloorItem.checked
  // 回显楼栋
  curBuildItem.checked = curBuildItem.list.every(e => e.checked)
  curSchoolItem.actived = curBuildItem.actived = curBuildItem.list.some(e => e.checked)

  // 优先-回显判断学校是否高亮
  callBackSchool()
  // 回显判断楼栋是否高亮
  callBackBuild()
}

// 回显楼栋是否选中,学校是否对应高亮
const callBackBuild = () => {
  const curSchoolItem = schoolList.find(e => e.schoolId == activeSchoolId.value)
  // 平铺判断楼栋楼层是否有选中
  const flatList = curSchoolItem.list.flatMap(e => e.list ? e.list : e)
  curSchoolItem.actived = flatList.some(e => e.checked)
}

// 回显学校是否选中
const callBackSchool = () => {
  const curSchoolItem = schoolList.find(e => e.schoolId == activeSchoolId.value)
  curSchoolItem.checked = curSchoolItem.list.every(e => e.checked)
}
</script>

<style lang="scss" scoped>
$active-color: #1b6aff;

// 图标展开动画
@keyframes iconExpand {
  0% {
    transform: rotateZ(0deg);
  }

  50% {
    transform: rotateZ(45deg);
  }

  100% {
    transform: rotateZ(90deg);
  }
}

// 图标关闭动画
@keyframes iconClose {
  0% {
    transform: rotateZ(90deg);
  }

  50% {
    transform: rotateZ(45deg);
  }

  100% {
    transform: rotateZ(0deg);
  }
}

.check-box {
  width: 100%;
  padding: 24px 0;
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
  user-select: none;

  .name,
  .active-name {
    margin-left: 8px
  }

  .active-name {
    color: $active-color;
  }
}

// 复选框样式
.checked-btn-box {
  width: 28px;
  height: 28px;
  position: absolute;
  right: 24px;
  top: 50%;
  transform: translateY(-50%);

  .unchecked {
    display: inline-block;
    border-radius: 4px;
    width: 28px;
    height: 28px;
    border: 1px solid $active-color;
    position: absolute;
  }

  .checked-btn {
    width: 30px;
    height: 30px;
    position: absolute;
    z-index: 2;
  }
}

// 通用样式
.school-box,
.build-box,
.floor-box {
  width: 702px;
  // padding: 0 24px;
  display: flex;
  align-items: center;
  flex-flow: column nowrap;
  background-color: #fff;
  cursor: pointer;
}

.school-item,
.build-item,
.floor-item {
  width: 100%;
  padding: 24px;
  position: relative;

  &::after {
    content: '';
    width: calc(100% - 48px);
    height: 1px;
    background-color: #caccd2;
    position: absolute;
    left: 24px;
    bottom: 1px;
  }
}

// 学校样式
.school-box {
  padding: 24px;
  margin: auto;
  align-items: start;
  border-radius: 16px;
  overflow: hidden;

  &:not(:last-child) {
    margin-bottom: 16px;
  }

}

.school-item {
  padding: 0;
  display: flex;
  align-items: center;

  &:not(:last-child)::after {
    content: '';
    height: 0px;
  }

  .icon,
  .active-icon {
    width: 30px;
    height: 30px;
  }

  .icon {
    animation: iconClose linear .2s;
  }

  .active-icon {
    transform: rotateZ(90deg);
    animation: iconExpand linear .2s;
  }
}

// 楼栋样式
.build-box {
  transition: all .2s linear;
}

.build-item {
  width: calc(100% - 48px);
  display: flex;
  align-items: center;
}

// 楼层样式
.floor-item {
  width: calc(100% - 48px);
}

.build-box:last-child .floor-box .floor-item:last-child {

  // 取消最后一楼下划线
  &::after {
    background-color: #fff;
  }
}
</style>
