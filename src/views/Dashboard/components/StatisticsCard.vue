<template>
  <el-card :style="{ backgroundColor: color + '10', borderLeft: `4px solid ${color}` }" class="statistics-card">
    <div class="card-content">
      <div class="card-header">
        <span class="card-title">{{ title }}</span>
        <el-icon :style="{ color }" class="card-icon">
          <component :is="iconComponent" />
        </el-icon>
      </div>
      <div class="card-value">{{ value }}</div>
      <div class="card-change" :class="{ up: change > 0, down: change < 0, flat: change === 0 }">
        <el-icon v-if="change > 0" size="14">
          <CaretTop />
        </el-icon>
        <el-icon v-if="change < 0" size="14">
          <CaretBottom />
        </el-icon>
        <span>{{ change === 0 ? '持平' : `${Math.abs(change)}%` }}</span>
        <span class="change-desc">较昨日</span>
      </div>
    </div>
  </el-card>
</template>

<script setup lang="ts">
// 修复图标导入路径（Element Plus 2.3.0+ 版本的正确导入方式）
import { CaretTop, CaretBottom } from '@element-plus/icons-vue'
import { User, Search, Document, Money } from '@element-plus/icons-vue'
import { computed } from 'vue'

// 定义props并使用
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  value: {
    type: [Number, String],
    required: true
  },
  change: {
    type: Number,
    required: true
  },
  icon: {
    type: String,
    required: true,
    validator: (value: string) => {
      // 验证传入的图标是否在允许的列表中
      return ['User', 'Search', 'Document', 'Money'].includes(value)
    }
  },
  color: {
    type: String,
    required: true
  },
  isAmount: {
    type: Boolean,
    default: false
  }
})

// 根据传入的icon名称获取对应的组件（修复未使用components的问题）
const iconComponent = computed(() => {
  const iconMap = {
    User,
    Search,
    Document,
    Money
  }
  return iconMap[props.icon as keyof typeof iconMap]
})
</script>

<style scoped>
.card-content {
  padding: 16px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.card-title {
  color: #666;
  font-size: 14px;
}

.card-icon {
  font-size: 24px;
}

.card-value {
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 8px;
}

.card-change {
  font-size: 14px;
  display: flex;
  align-items: center;
}

.up {
  color: #f53f3f;
}

.down {
  color: #00b42a;
}

.flat {
  color: #c6c866;
}

.change-desc {
  margin-left: 4px;
  color: #86909c;
}
</style>
